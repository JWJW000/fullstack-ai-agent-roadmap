# 搜索： getElement*, querySelector*

 当元素彼此靠的很近时 DOM 导航属性（navigation property）非常有用。如果不是，那该怎么办？如何去获取页面上的任意元素？
### document.getElementById或者只使用 id

如果一个元素有 id 特性（attribute），那我们就可以使用 document.getElementById(id)方法获取该元素，无论它在哪里。

例如：
```
<div id ="elem">
 <div id="elem-content">Element</div>
</div>

<script>
	let elem = document.getElementById('elem');
	
	elem.style.background = 'red';
</script>
```
此外，还有一个通过 id 命名的全局变量，它引用了元素：
```
<div id="elem">
	<div id="elem-content">Element</div>
</div>

<script>
	// elem是对带有 id ="elem" 的 DOM 元素的引用
	elem.style.background = 'red';
	// id="elem-content"内有连字符，所以它不能成为一个变量
	// ...但是我们可以通过使用方括号 window['elem-content']
```
....除非我们声明一个具有相同名称的 JavaScript 变量，否则它具有优先权：

```
<div id="elem"></div>

<script>
	let elem = 5;
	alert(elem); //5
```

> 请不要使用以 id 命名的全局变量来访问元素
> 浏览器尝试通过混合 JavaScript 和 DOM 的命名空间来帮助我们。对于内联到 HTML 中的简单脚本来说，这还行，但是通常来说，这不是一件好事。因为这可能会造成命名冲突。另外，当人们阅读 js 代码且看不到对应的 HTML 时，变量的来源就会不明显。在实际开发中 document.getElementById是首选方法。


>**id 必须是唯一的**
>如果有多个元素都带有同一个 id，那么使用它的方法的行为是不可预测的，例如 document.getElementById可能会随机返回其中一个元素。因此，请遵守规则，保持 id 的唯一性。

## querySelectorAll

到目前为止，最通用的方法是 elem.querySelectorAll(css),它返回 elem 中与给定 css 选择器匹配的所有元素。

在这里，我们查找所有为最后一个子元素的<li>元素：

```
<ul>
	<li>The</li>
	<li>test</li>
</ul>
<ul>
	<li>has</li>
	<li>passed</li>
</ul>
<script>
	let elements = document.querySelectorAll('ul > li:last-child');
	for(let elem of elements) {
		alert(elem.innerHTML); //"test", "passed"
	}
</script>
```

这个方法确实功能强大，因为可以使用任何 CSS 选择器。


也可以使用伪类

CSS 选择器的伪类，例如 `:hover` 和 `:active` 也都是被支持的。例如，`document.querySelectorAll(':hover')` 将会返回鼠标指针正处于其上方的元素的集合（按嵌套顺序：从最外层 `<html>` 到嵌套最多的元素）。

querySelector

`elem.querySelector(css)` 调用会返回给定 CSS 选择器的第一个元素。

换句话说，结果与 `elem.querySelectorAll(css)[0]` 相同，但是后者会查找 **所有** 元素，并从中选取一个，而 `elem.querySelector` 只会查找一个。因此它在速度上更快，并且写起来更短。


