title: 相册
noDate: 'true'
---
<link type="text/css" href="/fancybox/jquery.fancybox.css" rel="stylesheet">
<div class="instagram"><section class="archives album"><ul class="img-box-ul"></ul></section></div>
	
</div>
<script src="/js/photo.js"></script>
<script src="/js/jquery-3.1.1.min.js"></script>
<script type="text/javascript">
function test() {
	$.getJSON("/photo/output.json", function (data) {
		var li="";
    	 for(i=0;i<=data.length-1;i++){
    	 	li += '<div style="width:250px;height:250px;float:left;margin-left:10px"><img src="https://raw.githubusercontent.com/MaxieLj/blog/master/photos/' + data[i] + '"  style="width:100%; height:100%;float:left"/></div>'
                   ;
    	 }
       li+='<div style="clear:both"></div>'
    	 $('.instagram').append(li);   	
    });
    
}
test();

	
</script>