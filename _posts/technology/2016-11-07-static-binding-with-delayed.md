---
layout: post
title: 延迟静态绑定：static 关键字
category: 技术
tags: PHP
keywords: static 
description: 延迟静态绑定：static 关键字
---

### 开始

如果你是一位和我一样懒惰的程序员，当看到类似下面例子中的重复代码时，你可能会很恼火。
```php
abstract class A { }

class B extends A {
    public static function create () {
        return new B();
    }
}

class C extends A {
    public static function create(){
        return new C();
    } 
}
```

这段代码能够很好的工作，但是大量的重复的代码很烦人，我不想为每个子类都创造与这段类似的标准代码。因此为何不把 create() 放在超类中去呢？
```php
abstract class A{
    public static function create(){
        return new self();
    }
}

class B extends A{ }

class C extends A{ }

$test=B::create();
```
这样看起来是不是舒服多了？将重复的代码放在超类中去，并且假设 self 作为对该类的引用。实际上，self 指的不是上下文，它指的是解析上下文。因此，如果运行该段代码会得到：
    
    Fatal error: Uncaught Error: Cannot instantiate abstract class A in E:\Code\isBlog\static.php:4 Stack trace: #0 E:\Code\isBlog\static.php(12): A::create() #1 {main} thrown in E:\Code\isBlog\static.php on line 4


因此，self 被解析为定义 create() 的超类 A ，而不是解析为调用 self 的 B 类。

### static 关键字

PHP5.3 中引入了延迟静态绑定的概念，使用关键字 static。static 类似于 self ,不过它指的是被调用的类而不是包含它的类（即本例中的超类A）。
其实前面我们的意思是调用 B::create() 生成一个新的 B 对象，而不是试图实例化一个 A 对象。

因此，现在我们将利用 static 关键词来替换 self：

```php
abstract class A{
    public static function create(){
        return new static();
    }
}

class B extends A{ }

class C extends A{ }

var_dump($test=B::create());
```

ok ，程序正常运行，并得到了我们想要的结果。完美解决了我们一开始遇到的问题——大量重复的代码。

###  其他用法

static 关键字不仅仅可以用来实例化， 它还可以作为静态方法调用的标识符，甚至是从上下文中调用。

```php
abstract class A{
    private $data;
    public function __construct(){
        $this->data=static::getData();
    }
    public static function create(){
        return new static();
    }
    static function getData(){
        return "default";
    }
}

class B extends A{ }

class C extends A{
    static function getData(){
    return "C-object";    
    }
 }

class D extends C { }
var_dump(B::create());
var_dump(D::create());
```

超类 A 定义了构造方法，该构造方法使用 static 关键字调用静态方法getData()。A 类提供了默认的实现，但是 C 类将其覆盖了，D 类扩展 C类。
程序输出如下：

    object(B)#1 (1) { ["data":"A":private]=> string(7) "default" } 
    object(D)#1 (1) { ["data":"A":private]=> string(8) "C-object" }


### 结束

一次愉快的尝试。