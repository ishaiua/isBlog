---
layout: post
title: PHP Trait
category: 技术
tags: PHP Trait
description: PHP Trait
---

### Begin

由于最近在接触 [Laravel 框架](https://laravel.com/) 。发现里面很多地方使用 Trait 这个语法。于是乎对此查到了一些资料，作为笔记记录如下：

### Trait

自 PHP 5.4.0 起，PHP 实现了一种代码复用的方法，称为 trait。官方解释为：

> Trait 是为类似 PHP 的单继承语言而准备的一种代码复用机制。Trait 为了减少单继承语言的限制，使开发人员能够自由地在不同层次结构内独立的类中复用 method。Trait 和 Class 组合的语义定义了一种减少复杂性的方式，避免传统多继承和 Mixin 类相关典型问题。
Trait 和 Class 相似，但仅仅旨在用细粒度和一致的方式来组合功能。 无法通过 trait 自身来实例化。它为传统继承增加了水平特性的组合；也就是说，应用的几个 Class 之间不需要继承。

简单的说，就是将重复的代码拆分到一个文件中，用过 use 关键字引入以达到代码复用的目的。

### 简单示例：

```php
<?php
trait RunTrait{
	public function run(){
		echo __CLASS__." is running……";
	}
}

class Dog{
	use RunTrait;
}

class Cat{
	use RunTrait;
}

$dog=new Dog();
$dog->run();   //Dog is running……
$cat=new Cat();
$cat->run();  //Cat is running……
?>
```
关于何时使用 Trait 可以参考 安正超写的 [我所理解的 PHP Trait](http://overtrue.me/articles/2016/04/about-php-trait.html)。

### 优先级

从基类继承的成员会被 trait 插入的成员所覆盖。优先顺序是来自当前类的成员覆盖了 trait 的方法，而 trait 则覆盖了被继承的方法。

官方示例：

```php
<?php
class Base {
    public function sayHello() {
        echo 'Hello ';
    }
}

trait SayWorld {
    public function sayHello() {
        parent::sayHello();
        echo 'World!';
    }
}

class MyHelloWorld extends Base {
    use SayWorld;
}

$o = new MyHelloWorld();
$o->sayHello();   //Hello World!
?>
```

### 多个 Trait ###

通过逗号分隔，在 use 声明列出多个 trait，可以都插入到一个类中。

如果两个 trait 都插入了一个同名的方法，如果没有明确解决冲突将会产生一个致命错误。

为了解决多个 trait 在同一个类中的命名冲突，需要使用 insteadof 操作符来明确指定使用冲突方法中的哪一个。

以上方式仅允许排除掉其它方法，as 操作符可以将其中一个冲突的方法以另一个名称来引入。

示例：

```php
<?php
trait A {
    public function run() {
        echo 'Trait A running...';
    }
    
}

trait B {
    public function run() {
        echo 'Trait B running...';
    }
}

class Dog {
    use A, B {
        B::run insteadof A; 
    }
}

class Cat {
    use A, B {
        A::run insteadof B;
        B::run as runB;
    }
}

$dog=new Dog();
$dog->run();   //Trait B running...
$cat=new Cat();
$cat->runB();  //Trait B running...
?>
```


### 参考

1. [官方文档（中文）](http://www.php.net/manual/zh/language.oop5.traits.php)。
2. 安正超 [我所理解的 PHP Trait](http://overtrue.me/articles/2016/04/about-php-trait.html)。