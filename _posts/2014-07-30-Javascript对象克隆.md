---
layout: article
title: Javascript对象克隆
---

最近工作终于稍微闲了一点，把一些项目中用到的java和js基础重又复习了一下。在项目开发，因为使用前端或者后端的类库很多，致使用了很多功能，但原理却知之甚少。

JS的对象克隆，在开发中经常会用到，借助jQuery等类库可以很容易实现。

这里提供两种实现的方法，网上还可以搜到一些其他的实现方法。

第一种：针对对象的属性逐个逐层的复制。

    function clone(obj) {
		// Handle the 3 simple types, string\number\bool and null or undefined
		if (null == obj || "object" != typeof obj)
			return obj;

		// Handle Date
		if (obj instanceof Date) {
			var copy = new Date();
			copy.setTime(obj.getTime());
			return copy;
		}

		// Handle Array
		if (obj instanceof Array) {
			var copy = [];
			for ( var i = 0, len = obj.length; i < len; i++) {
				copy[i] = clone(obj[i]);
			}
			return copy;
		}

		// Handle Object
		if (obj instanceof Object) {
			var copy = {};
			for ( var attr in obj) {
				if (obj.hasOwnProperty(attr))
					copy[attr] = clone(obj[attr]);
			}
			return copy;
		}

		throw new Error("Unable to copy obj! Its type isn't supported.");
	}

第二种：将对象转为字符串，再将字符串转为一个新的对象。(这种方法在正确性和性能上都差一些，后面会有测试，这种方法不建议使用！)

    function copy(obj) {
		return JSON.parse(JSON.stringify(obj));
	}

测试代码：

    var a = {
		a : [ "array", {
			b : "data"
		} ]
	};

	console.log('第一种克隆方法：');
	console.log(JSON.stringify(a));
	var b = clone(a);
	console.log(JSON.stringify(b));

	console.log('修改源对象的值：');
	a.a[1].b = "test";
	console.log(JSON.stringify(a));
	console.log(JSON.stringify(b));

	a = {
		a : [ "array", {
			b : "data"
		} ]
	};
	console.log('第二种克隆方法：');
	console.log(JSON.stringify(a));
	b = copy(a);
	console.log(JSON.stringify(b));

	console.log('修改源对象的值：');
	a.a[1].b = "test";
	console.log(JSON.stringify(a));
	console.log(JSON.stringify(b));

	a = {
		a : [ "array", {
			b : "data"
		} ],
		d : new Date(),
		c : {
			a : '11',
			d : 'aaaa'
		}
	};
	console.log('测试两种方法的效率，复制十万个对象：');
	console.log('1:' + new Date().getTime());
	for ( var i = 0; i < 100000; i++) {
		b = clone(a);
	}
	console.log('2:' + new Date().getTime());
	for ( var i = 0; i < 100000; i++) {
		b = copy(a);
	}
	console.log('3:' + new Date().getTime());

运行结果：

![对象复制](/img/javascript-object-clone.jpg)