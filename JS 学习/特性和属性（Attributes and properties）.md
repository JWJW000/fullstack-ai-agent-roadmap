

当浏览器加载页面时，它会“读取” HTML 并从中生成 DOM 对象。对于元素节点，大多数标准的 HTML 特性会自动变成 DOM 对象的属性。

例如，如果标签是`<body id = "page">` ,那么DOM 对象就会有 body.id = "page"

但特性-属性映射并不是一一对应的！

# DOM属性

DOM节点是常规的JavaScript对象。我们可以更改它们。

例如，让我们在document.body中创建一个新的属性：

```
document.body.myDate = {
	name: 'Caesar',
	title: 'Imperator'
};

alert(document.body.myData.title);
```

我们也可以像下面这样添加一个方法：
```
document.body.sayTagname = function() {
	alert(this.tagName);
};

document.body.sayTagName(); // BODY
```

我们还可以修改内建属性的原型，例如修改 Element.protitype