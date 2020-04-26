## jQuery Select

jQuery 选择器允许对HTML元素组或者单个元素进行操作

- 元素选择器

~~~ javascript
$(document).ready(function(){
   $("div").click(function(){
       $("p").hide();
   }); 
});
~~~

* id 选择器

~~~ javascript
$(function(){
	$("button").click(function(){
		$("#test").hide();
	});
});
~~~

- class 选择器

~~~ javascript
$(function(){
	$("button").click(function(){
		$(".test").hide();
	});
});
~~~

- 全选(*)

~~~ javascript
$(function(){
	$("button").click(function(){
    	$("*").hide();
 	});
});
~~~

- 其他选择器

| 语法                     | 描述                                         |
| ------------------------ | -------------------------------------------- |
| $(this)a                 | 选取当前HTML元素                             |
| $("a.class")             | 选取class="class"的a元素                     |
| $("a:first")             | 选取第一个a元素                              |
| $("ul li:first")         | 选取第一个ul元素的第一个li元素               |
| $("ul li:first-child")   | 选取每个ul元素的第一个li元素                 |
| $("[src]")               | 选取带有src属性的元素                        |
| $("a[target='_blank']")  | 选取所有target的值为"_blank"的a元素          |
| $("a[target!='_blank']") | 选取所有target的值不为"_blank"的a元素        |
| $(":button")             | 选取所有type="button"的input元素和button元素 |
| $("tr:even")             | 选取偶数位的tr元素                           |
| $("tr:odd")              | 选取奇数位的tr元素                           |



