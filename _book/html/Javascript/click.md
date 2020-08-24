## jquery 事件

- click()  点击触发事件

  ~~~ javascript
  $(function () {
      $("button").click(function () {
		console.log("点击事件");
      });
  });
  ~~~
  
- dblclick()  双击元素触发事件

  ~~~ javascript
  $(function () {
  	$("button").dblclick(function () {
  		console.log("双击事件");
  	});
  });
  ~~~

- mouseenter() 鼠标放到元素上触发事件

  ~~~ javascript
  $(function () {
  	$("button").mouseenter(function () {
  		console.log("鼠标穿过元素");
  	});
  });
  ~~~

- mouseleave() 鼠标离开元素时触发事件

  ~~~ javascript
  $(function () {
  	$("button").mouseleave(function () {
  		console.log("鼠标离开事件");		
  	});
  });
  ~~~

- mouseup() 当元素上的鼠标松开时触发事件

  ~~~ javascript
  $(function () {
  	$("button").mouseup(function () {
  		console.log("鼠标松开时触发事件");
  	});
  });
  ~~~

- mousedown() 当鼠标按下时触发事件

  ~~~ javascript
  $(function () {
  	$("button").mousedown(function () {
  		console.log("鼠标按下时触发事件");
  	});
  });
  ~~~

- hover() 当鼠标悬停在元素上触发事件

  ~~~ javascript
  $(function () {
  	$("button").hover(function () {
  		console.log("鼠标悬停在元素上事件");
  	});
  });
  ~~~

- focus()  当元素获取焦点时触发事件

  ~~~ javascript
  $(function () {
  	$("input#test").focus(function () {
  		console.log("获取焦点事件");
  	});
  });
  ~~~

- blur()  当元素失去焦点时触发事件

  ~~~ javascript
  $(function () {
  	$("input#test").blur(function () {
  		console.log("元素失去焦点时触发的事件");
  	})
  })
  ~~~

  

