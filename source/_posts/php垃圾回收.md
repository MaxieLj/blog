---
title: php垃圾回收
date: 2018-09-12 21:28:00
tags: php源码
categories: php源码学习
toc: true
---

php是如何实现内存管理的?内存管理无非包括内存分配、内存回收、以及内存使用优化。

- 内存使用的优化
- 垃圾回收机制
- 底层内存分配


## 内存使用的优化

### 引用计数

php的引用中有个引用结构体

```php
```c
struct _zend_reference {
    zend_refcondted_h gc;
    zval              val;  指向原来的value.
};
```
```
其中`zend_refcondted_h` 便是gc便是当前变量被引用的次数,这个参数会在变量回收的时候用到。


zend_refcondted_h :

```c
typedef struct _zend_refcounted_h {
	uint32_t         refcount;			/* reference counter 32-bit */
	union {
		struct {
			ZEND_ENDIAN_LOHI_3(
				zend_uchar    type,
				zend_uchar    flags,    /* used for strings & objects */
				uint16_t      gc_info)  /* keeps GC root number (or 0) and color */
		} v;
		uint32_t type_info;
	} u;
} zend_refcounted_h;
```

在实际中这个结构体到底是什么样的? 具体可以举例来看。

```php
$a = 'this is string'; // zend_array (refcount = 1)  只有$a引用了zend_array
$b = &$a; //   zend_array (refcount = 2)  $a、$b引用了zend_array
$c = $b; // zend_array (refcount = 3)  $a、$b、$c引用了zend_array
unset($b); // zend_array (refcount = 2)  $a、$c引用了zend_array
```

> 并不是所有的变量类型都会使用引用计数, 例如 `整形`、`浮点型`、`布尔型`、`NUll`(在php中这是一个变量类型)等采用了深拷贝,
即只要这几种变量赋值, 就会申请一款一块内存写入一个新的变量结构（注意这里不是引用结构）,当然这些类型不会公用value。



### 写时复制

当然只有引用计数是不够的, 因为变量会发生赋值的情况。所以在更改某一个变量时, 会对原来的变量进行拷贝并赋值。

举个栗子:

```
$foo = time();
$bar = &$b;
$si = $a;

$c = '123';
```

具体数据结构的引用计数情况如下图:



![image](/photo/img/php内存管理/写时复制.png)

## 内存回收

### 自动gc
在zend数据接口中有一个gc.refount,他是自动gc的关键。

在自动gc机制中,如果zval不在指向value且当前value的gc.refount为0时,会直接释放value。这种情况多发生于函数返回时（销毁所有局部变量）、变量修改时、以及unset操作的时候。


### 辣鸡回收

除了自动gc,还有一种是自动gc无法处理的垃圾, 这种情况称为`循环引用`。顾名思义也就是自身内部变量引用了自身, 这种情况常出现与object和array。当我该变量zval被销毁是, 与其对应的value gc.refcount -1,
但是因为有自身变量指向自身, 所以就陷入了一个循环:如果不销毁自身变量 value gc.refcount就无法自动gc, 只有自动gc才会销毁value。

```php

$a = [1];
$a[] = &$a;
unset($a);

```

![image](/photo/img/php内存管理/自身引用.png)

`unset($a)`执行以后

![image](/photo/img/php内存管理/释放.png)

由于上述情况导致无法自动gc,所以需要引入另外一钟垃圾回收机制-垃圾回收器。
现在会存在两种情况的数据需要回收：
- 当value的gc.refcount =0 是需要回收。
- 当value的gc.refcount 减少不等于0，但是存在循环引用时。

### 回收机制

当发生value gc.refcount减少时，垃圾回收机制会把可能是垃圾的value存起来， 当可能是辣鸡的value到达一定数量的时候启动垃圾鉴别程序，统一处理垃圾。当然这里的value类型只有object和array。

垃圾兼备程序：
其实垃圾鉴别程序很简单，递归遍历自身value，查看是否存在指向自身的即可。

code:
gc 初始化
```php
ZEND_API void gc_init(void)
{
	if (GC_G(buf) == NULL && GC_G(gc_enabled)) {
		//初始化buf内存区域，大小为sizeof(gc_root_buffer) * //GC_ROOT_BUFFER_MAX_ENTRIES
		GC_G(buf) = (gc_root_buffer*) malloc(sizeof(gc_root_buffer) * GC_ROOT_BUFFER_MAX_ENTRIES);
		//设置_zend_gc_globals.last_unused为bug入口位置
		GC_G(last_unused) = &GC_G(buf)[GC_ROOT_BUFFER_MAX_ENTRIES];
		//初始化_zend_gc_globals的参数
		gc_reset();
	}
}
```
垃圾回收及其依赖 `_zend_gc_globals`

`_zend_gc_globals`
```c
typedef struct _zend_gc_globals {
	zend_bool         gc_enabled;
	zend_bool         gc_active;
	zend_bool         gc_full;

	gc_root_buffer   *buf;				/* preallocated arrays of buffers   */
	gc_root_buffer    roots;			/* list of possible roots of cycles */
	gc_root_buffer   *unused;			/* list of unused buffers           */
	gc_root_buffer   *first_unused;		/* pointer to first unused buffer   */
	gc_root_buffer   *last_unused;		/* pointer to last unused buffer    */

	gc_root_buffer    to_free;			/* list to free                     */
	gc_root_buffer   *next_to_free;

	uint32_t gc_runs;
	uint32_t collected;

#if GC_BENCH
	uint32_t root_buf_length;
	uint32_t root_buf_peak;
	uint32_t zval_possible_root;
	uint32_t zval_buffered;
	uint32_t zval_remove_from_buffer;
	uint32_t zval_marked_grey;
#endif

	gc_additional_buffer *additional_buffer;

} zend_gc_globals;
```

- `gc_enabled` 是否使使用gc
- `gc_active`  是否在垃圾检查的过程中
- `gc_full` buf缓冲区是否已满
- `*buf` 与分配用于保存可能为垃圾的value
- `roots` 指向buf最新加入的一个可能垃圾
- `unused` 指向第未使用的buffer
- `*first_unused` 指向第一个没用使用buffer
- `*last_unused` 指向buffer的尾部
- `to_free` 等待释放的buffer
- `gc_runs`  统计gc运行的次数
- `collected`  统计已经释放的垃圾数


php垃圾回收中几个重要的颜色写在zeng_gc的备注中。
```C
 * BLACK  (GC_BLACK)   - In use or free.
 * GREY   (GC_GREY)    - Possible member of cycle.
 * WHITE  (GC_WHITE)   - Member of garbage cycle.
 * PURPLE (GC_PURPLE)  - Possible root of cycle.
```
- GC_WHITE 白色表示垃圾
- GC_PURPLE 紫色表示已放入缓冲区
- GC_GREY 灰色表示已经进行了一次refcount的减一操作
- GC_BLACK 黑色是默认颜色，正常


gc过程中主要处理功能的函数`zend_gc_collect_cycles`

```c
ZEND_API void ZEND_FASTCALL gc_possible_root(zend_refcounted *ref)
{
	gc_root_buffer *newRoot;

	if (UNEXPECTED(CG(unclean_shutdown)) || UNEXPECTED(GC_G(gc_active))) {
		return;
	}

	ZEND_ASSERT(GC_TYPE(ref) == IS_ARRAY || GC_TYPE(ref) == IS_OBJECT);
	ZEND_ASSERT(EXPECTED(GC_REF_GET_COLOR(ref) == GC_BLACK));
	ZEND_ASSERT(!GC_ADDRESS(GC_INFO(ref)));

	GC_BENCH_INC(zval_possible_root);

	newRoot = GC_G(unused);
	if (newRoot) {
		GC_G(unused) = newRoot->prev;
	} else if (GC_G(first_unused) != GC_G(last_unused)) {
		newRoot = GC_G(first_unused);
		GC_G(first_unused)++;
	} else {
		//垃圾处理，当缓冲区满时，程序调用gc_collect_cycles函数，执行垃圾回收操作。 其中最关键的几步就是
		//如果当前处于可以gc的状态
		if (!GC_G(gc_enabled)) {
			return;
		}
		GC_REFCOUNT(ref)++;
		//垃圾回收
		gc_collect_cycles();
		GC_REFCOUNT(ref)--;
		if (UNEXPECTED(GC_REFCOUNT(ref)) == 0) {
			zval_dtor_func(ref);
			return;
		}
		if (UNEXPECTED(GC_INFO(ref))) {
			return;
		}
		newRoot = GC_G(unused);
		if (!newRoot) {
			return;
		}
		GC_G(unused) = newRoot->prev;
	}

	GC_TRACE_SET_COLOR(ref, GC_PURPLE);
	GC_INFO(ref) = (newRoot - GC_G(buf)) | GC_PURPLE;
	newRoot->ref = ref;

	newRoot->next = GC_G(roots).next;
	newRoot->prev = &GC_G(roots);
	GC_G(roots).next->prev = newRoot;
	GC_G(roots).next = newRoot;

	GC_BENCH_INC(zval_buffered);
	GC_BENCH_INC(root_buf_length);
	GC_BENCH_PEAK(root_buf_peak, root_buf_length);
}
```


debug代码已删除
1. 深度优先对对象或者数据的每一个元素的`refcount--`并将其标记为灰色
2. 深度遍历root的每个每个变量，如果此时变量的`refcount`为0，则代表着改变量为垃圾，将其标记为垃圾，如果不为0，则将其标记为黑色（正常）。
3. 检查roots清除标记为白色的垃圾。

//TODO 垃圾回收抽出来出来写。

具体代码：
```php
ZEND_API int zend_gc_collect_cycles(void)
{
	int count = 0;

	if (GC_G(roots).next != &GC_G(roots)) {
		gc_root_buffer *current, *next, *orig_next_to_free;
		zend_refcounted *p;
		gc_root_buffer to_free;
		uint32_t gc_flags = 0;
		gc_additional_buffer *additional_buffer_snapshot;


		if (GC_G(gc_active)) {
			return 0;
		}

		GC_TRACE("Collecting cycles");
		//标识gc运行了多少次
		GC_G(gc_runs)++;
		//标识当前正在gc
		GC_G(gc_active) = 1;

		GC_TRACE("Marking roots");
		//重点
		gc_mark_roots();
		GC_TRACE("Scanning roots");
		//重点
		gc_scan_roots();



		GC_TRACE("Collecting roots");
		additional_buffer_snapshot = GC_G(additional_buffer);
		count = gc_collect_roots(&gc_flags);

		GC_G(gc_active) = 0;

		if (GC_G(to_free).next == &GC_G(to_free)) {
			/* nothing to free */
			GC_TRACE("Nothing to free");
			return 0;
		}

		/* Copy global to_free list into local list */
		to_free.next = GC_G(to_free).next;
		to_free.prev = GC_G(to_free).prev;
		to_free.next->prev = &to_free;
		to_free.prev->next = &to_free;

		/* Free global list */
		GC_G(to_free).next = &GC_G(to_free);
		GC_G(to_free).prev = &GC_G(to_free);

		orig_next_to_free = GC_G(next_to_free);


		if (gc_flags & GC_HAS_DESTRUCTORS) {
			GC_TRACE("Calling destructors");

			/* Remember reference counters before calling destructors */
			current = to_free.next;
			while (current != &to_free) {
				current->refcount = GC_REFCOUNT(current->ref);
				current = current->next;
			}

			/* Call destructors */
			current = to_free.next;
			while (current != &to_free) {
				p = current->ref;
				GC_G(next_to_free) = current->next;
				if (GC_TYPE(p) == IS_OBJECT) {
					zend_object *obj = (zend_object*)p;

					if (!(GC_FLAGS(obj) & IS_OBJ_DESTRUCTOR_CALLED)) {
						GC_TRACE_REF(obj, "calling destructor");
						GC_FLAGS(obj) |= IS_OBJ_DESTRUCTOR_CALLED;
						if (obj->handlers->dtor_obj
						 && (obj->handlers->dtor_obj != zend_objects_destroy_object
						  || obj->ce->destructor)) {
							GC_REFCOUNT(obj)++;
							obj->handlers->dtor_obj(obj);
							GC_REFCOUNT(obj)--;
						}
					}
				}
				current = GC_G(next_to_free);
			}

			/* Remove values captured in destructors */
			current = to_free.next;
			while (current != &to_free) {
				GC_G(next_to_free) = current->next;
				if (GC_REFCOUNT(current->ref) > current->refcount) {
					gc_remove_nested_data_from_buffer(current->ref, current);
				}
				current = GC_G(next_to_free);
			}
		}

		/* Destroy zvals */
		GC_TRACE("Destroying zvals");
		GC_G(gc_active) = 1;
		current = to_free.next;
		while (current != &to_free) {
			p = current->ref;
			GC_G(next_to_free) = current->next;
			GC_TRACE_REF(p, "destroying");
			if (GC_TYPE(p) == IS_OBJECT) {
				zend_object *obj = (zend_object*)p;

				EG(objects_store).object_buckets[obj->handle] = SET_OBJ_INVALID(obj);
				GC_TYPE(obj) = IS_NULL;
				if (!(GC_FLAGS(obj) & IS_OBJ_FREE_CALLED)) {
					GC_FLAGS(obj) |= IS_OBJ_FREE_CALLED;
					if (obj->handlers->free_obj) {
						GC_REFCOUNT(obj)++;
						obj->handlers->free_obj(obj);
						GC_REFCOUNT(obj)--;
					}
				}
				SET_OBJ_BUCKET_NUMBER(EG(objects_store).object_buckets[obj->handle], EG(objects_store).free_list_head);
				EG(objects_store).free_list_head = obj->handle;
				p = current->ref = (zend_refcounted*)(((char*)obj) - obj->handlers->offset);
			} else if (GC_TYPE(p) == IS_ARRAY) {
				zend_array *arr = (zend_array*)p;

				GC_TYPE(arr) = IS_NULL;

				/* GC may destroy arrays with rc>1. This is valid and safe. */
				HT_ALLOW_COW_VIOLATION(arr);

				zend_hash_destroy(arr);
			}
			current = GC_G(next_to_free);
		}

		/* Free objects */
		current = to_free.next;
		while (current != &to_free) {
			next = current->next;
			p = current->ref;
			if (EXPECTED(current >= GC_G(buf) && current < GC_G(buf) + GC_ROOT_BUFFER_MAX_ENTRIES)) {
				current->prev = GC_G(unused);
				GC_G(unused) = current;
			}
			efree(p);
			current = next;
		}

		while (GC_G(additional_buffer) != additional_buffer_snapshot) {
			gc_additional_buffer *next = GC_G(additional_buffer)->next;
			efree(GC_G(additional_buffer));
			GC_G(additional_buffer) = next;
		}

		GC_TRACE("Collection finished");
		GC_G(collected) += count;
		GC_G(next_to_free) = orig_next_to_free;
		GC_G(gc_active) = 0;
	}

	return count;
}
```


```c
static void gc_mark_roots(void)
{
	gc_root_buffer *current = GC_G(roots).next;

	while (current != &GC_G(roots)) {
		//GC_PURPLE 标识在缓冲区
		if (GC_REF_GET_COLOR(current->ref) == GC_PURPLE) {
			gc_mark_grey(current->ref);
		}
		current = current->next;
	}
}
```

```c
static void gc_mark_grey(zend_refcounted *ref)
{
    HashTable *ht;
	Bucket *p, *end;
	zval *zv;

tail_call:
	if (GC_REF_GET_COLOR(ref) != GC_GREY) {
		ht = NULL;
		GC_BENCH_INC(zval_marked_grey);
		GC_REF_SET_COLOR(ref, GC_GREY);

		if (GC_TYPE(ref) == IS_OBJECT) {
			zend_object_get_gc_t get_gc;
			zend_object *obj = (zend_object*)ref;

			if (EXPECTED(!(GC_FLAGS(ref) & IS_OBJ_FREE_CALLED) &&
		                 (get_gc = obj->handlers->get_gc) != NULL)) {
				int n;
				zval *zv, *end;
				zval tmp;

				ZVAL_OBJ(&tmp, obj);
				ht = get_gc(&tmp, &zv, &n);
				end = zv + n;
				if (EXPECTED(!ht)) {
					if (!n) return;
					while (!Z_REFCOUNTED_P(--end)) {
						//表明当前object size为0
						if (zv == end) return;
					}
				}
				while (zv != end) {
					//循环对每个元素进行--
					if (Z_REFCOUNTED_P(zv)) {
						ref = Z_COUNTED_P(zv);
						GC_REFCOUNT(ref)--;
						//refcount已经减过，标记为灰色
						gc_mark_grey(ref);
					}
					zv++;
				}
				if (EXPECTED(!ht)) {
					ref = Z_COUNTED_P(zv);
					GC_REFCOUNT(ref)--;
					goto tail_call;
				}
			} else {
				return;
			}
		} else if (GC_TYPE(ref) == IS_ARRAY) {
			if (((zend_array*)ref) == &EG(symbol_table)) {
				//标识是正常非垃圾
				GC_REF_SET_BLACK(ref);
				return;
			} else {
				ht = (zend_array*)ref;
			}
		} else if (GC_TYPE(ref) == IS_REFERENCE) {
			if (Z_REFCOUNTED(((zend_reference*)ref)->val)) {
				ref = Z_COUNTED(((zend_reference*)ref)->val);
				GC_REFCOUNT(ref)--;
				goto tail_call;
			}
			return;
		} else {
			return;
		}

		if (!ht->nNumUsed) return;
		p = ht->arData;
		end = p + ht->nNumUsed;
		while (1) {
			end--;
			zv = &end->val;
			if (Z_TYPE_P(zv) == IS_INDIRECT) {
				zv = Z_INDIRECT_P(zv);
			}
			if (Z_REFCOUNTED_P(zv)) {
				break;
			}
			if (p == end) return;
		}
		while (p != end) {
			zv = &p->val;
			if (Z_TYPE_P(zv) == IS_INDIRECT) {
				zv = Z_INDIRECT_P(zv);
			}
			if (Z_REFCOUNTED_P(zv)) {
				ref = Z_COUNTED_P(zv);
				GC_REFCOUNT(ref)--;
				gc_mark_grey(ref);
			}
			p++;
		}
		zv = &p->val;
		if (Z_TYPE_P(zv) == IS_INDIRECT) {
			zv = Z_INDIRECT_P(zv);
		}
		ref = Z_COUNTED_P(zv);
		GC_REFCOUNT(ref)--;
		goto tail_call;
	}
}
```

```c
static void gc_scan(zend_refcounted *ref)
{
    HashTable *ht;
	Bucket *p, *end;
	zval *zv;

tail_call:
	if (GC_REF_GET_COLOR(ref) == GC_GREY) {
		if (GC_REFCOUNT(ref) > 0) {
			//所有refount--以后如果还>0，说明非垃圾
			gc_scan_black(ref);
		} else {
			//否则则为垃圾
			GC_REF_SET_COLOR(ref, GC_WHITE);
			if (GC_TYPE(ref) == IS_OBJECT) {
				zend_object_get_gc_t get_gc;
				zend_object *obj = (zend_object*)ref;

				if (EXPECTED(!(GC_FLAGS(ref) & IS_OBJ_FREE_CALLED) &&
				             (get_gc = obj->handlers->get_gc) != NULL)) {
					int n;
					zval *zv, *end;
					zval tmp;

					ZVAL_OBJ(&tmp, obj);
					ht = get_gc(&tmp, &zv, &n);
					end = zv + n;
					if (EXPECTED(!ht)) {
						if (!n) return;
						while (!Z_REFCOUNTED_P(--end)) {
							if (zv == end) return;
						}
					}
					while (zv != end) {
						if (Z_REFCOUNTED_P(zv)) {
							ref = Z_COUNTED_P(zv);
							gc_scan(ref);
						}
						zv++;
					}
					if (EXPECTED(!ht)) {
						ref = Z_COUNTED_P(zv);
						goto tail_call;
					}
				} else {
					return;
				}
			} else if (GC_TYPE(ref) == IS_ARRAY) {
				if ((zend_array*)ref == &EG(symbol_table)) {
					GC_REF_SET_BLACK(ref);
					return;
				} else {
					ht = (zend_array*)ref;
				}
			} else if (GC_TYPE(ref) == IS_REFERENCE) {
				if (Z_REFCOUNTED(((zend_reference*)ref)->val)) {
					ref = Z_COUNTED(((zend_reference*)ref)->val);
					goto tail_call;
				}
				return;
			} else {
				return;
			}

			if (!ht->nNumUsed) return;
			p = ht->arData;
			end = p + ht->nNumUsed;
			while (1) {
				end--;
				zv = &end->val;
				if (Z_TYPE_P(zv) == IS_INDIRECT) {
					zv = Z_INDIRECT_P(zv);
				}
				if (Z_REFCOUNTED_P(zv)) {
					break;
				}
				if (p == end) return;
			}
			while (p != end) {
				zv = &p->val;
				if (Z_TYPE_P(zv) == IS_INDIRECT) {
					zv = Z_INDIRECT_P(zv);
				}
				if (Z_REFCOUNTED_P(zv)) {
					ref = Z_COUNTED_P(zv);
					gc_scan(ref);
				}
				p++;
			}
			zv = &p->val;
			if (Z_TYPE_P(zv) == IS_INDIRECT) {
				zv = Z_INDIRECT_P(zv);
			}
			ref = Z_COUNTED_P(zv);
			goto tail_call;
		}
	}
}
```
主要为三个函数：
- `gc_mark_roots`队规遍历，对object、array所有元素的refcount--并将其标记为灰色
- `gc_scan_roots`这个函数对节点信息的链表再次进行深度优先遍历，如果发现zval的refcount大于等于1，则对该zval和其包含的zval的refcount加1操作，这个是对非垃圾的一个信息还原，然后将这些zval颜色属性去掉(设置成black)。如果发现zval的refcount等于0，则就标记成白色，这些是后面将要清理掉的垃圾。
- `gc_collect_roots` 遍历节点信息链表,将前面一个步骤中标记为白色的节点信息放到GC_G(zval_to_free)为入口的链表中，这个链表用来存放将要释放的垃圾。 然后释放掉全部的节点信息，缓冲区被清空。分析结束后将重新收集节点信息。

