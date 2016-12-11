---
layout: post
title: PHP 反射 API
category: 技术
tags: PHP Reflection API
description: PHP Reflection
---

### 简介

> PHP5 具有完整的反射 API，添加了对类、接口、函数、方法和扩展进行反向工程的能力。 此外，反射 API 提供了方法来取出函数、类和方法中的文档注释。 --[官方文档（中文）](http://php.net/manual/zh/book.reflection.php)

### 安装

无需安装，其为 PHP 核心的一部分。

### 反射 API 的部分类


<table class="table table-bordered table-striped table-condensed" width="80%;">
    <tr>
        <td width="40%;">类</td><td>摘要</td>
    </tr>
    <tr>
    <td>Reflection</td><td>为类的摘要信息提供静态函数 export() </td>
    </tr>
    <tr>
    <td>ReflectionClass</td><td>类信息和工具</td>
    </tr>
    <tr>
    <td>ReflectionMethod</td><td>顾名思义类方法信息和工具</td>
    </tr>
    <tr>
    <td>ReflectionParaneter</td><td>方法参数信息</td>
    </tr>
    <tr>
    <td>ReflectionProperty</td><td>类属性信息</td>
    </tr>
    <tr>
    <td>ReflectionFunction</td><td>函数信息和工具</td>
    </tr>
    <tr>
    <td>ReflectoonExtension</td><td>PHP扩展信息</td>
    </tr>
     <tr>
    <td>ReflectoonException</td><td>异常类</td>
    </tr>
</table>

### Reflection::export()

示例：


```php
<?php
class Animal{
	private $name;

	public function __construct($name){
		$this->name=$name;
	}

	protected function run(){
		echo $this->name.' is running...';
	}

}
$animal=new Animal('cat');
$ref=new ReflectionClass($animal);
echo "<pre>";
Reflection::export($ref);
?>
```
运行代码可以得到：

```php
Class [  class Animal ] {
  @@ E:\Code\isBlog\reflection.php 2-13

  - Constants [0] {
  }

  - Static properties [0] {
  }

  - Static methods [0] {
  }

  - Properties [1] {
    Property [  private $name ]
  }

  - Methods [2] {
    Method [  public method __construct ] {
      @@ E:\Code\isBlog\reflection.php 5 - 7

      - Parameters [1] {
        Parameter #0 [  $name ]
      }
    }

    Method [  protected method run ] {
      @@ E:\Code\isBlog\reflection.php 9 - 11
    }
  }
}

```
我们可以看到 Reflection::export() 可以提供类几乎所有的信息。包括属性和方法的访问控制权限、每个方法需要的参数甚至代码在文件中的位置信息。

### 总结

更多 PHP API 的使用方法可以通过阅读 [官方文档（中文）](http://php.net/manual/zh/book.reflection.php) 获得。







