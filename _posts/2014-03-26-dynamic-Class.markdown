---
layout: post
title:  动态类温习
date:   2014-03-26 10:27:00
categories: study
---

“动态”类定义在运行时可通过添加/更改属性和方法来改变的对象。非动态类（如 String类）是“密封”类。不能在运行时向密封类中添加属性或方法。

在声明类时，可以通过使用 dynamic 属性来创建动态类 。
{% highlight actionscript %}
package
{
	
	/**
	 * 
	 * @author LC
	 */
	dynamic public class Person
	{
		
		public var name:String;
		
		public var age:int;
		
		public function Person()
		{
			trace("init person");
		}
	}
}
{% endhighlight %}
如果要在以后实例化 Person 类的实例，则可以在类定义的外部向该类中添加属性或方法 。
{% highlight actionscript %}
var person:Person = new Person();
trace("sex="+person.sex);
person.sex = "man";
trace("sex="+person.sex);
//打印信息
init person
sex=undefined
sex=man
{% endhighlight %}


但是，以这种方式创建的方法对于 Person 类的任何私有属性或方法都不具有访问权限。而且，即使对 Protean 类的公共属性或方法的引用也必须用 this 关键字或当前的实例进行限定。
{% highlight actionscript %}
var person:Person = new Person();
person.username = "xiaoli";
person.age = 30;
trace("sex="+person.sex);
person.sex = "man";
trace("sex="+person.sex);

person.run = function():void
{
	trace(person.username);
	trace(this.username);
	trace(person.sex);
	trace(this.sex);
}
person.run();

//打印信息
init person
sex=undefined
sex=man
xiaoli
xiaoli
man
man
{% endhighlight %}


平时用的Object就是一个动态类。MovieClip也是动态类,public dynamic class MovieClip

参考资料
<a href="http://bal1212.iteye.com/blog/978556" target="_blank">http://bal1212.iteye.com/blog/978556</a>