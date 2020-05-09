---
layout: post
title: PHP类反射
tags:
  - php
categories:
  - php
abbrlink: 46494
date: 2019-06-11 12:07:06
---

### 前言

面向对象编程中对象被赋予了自省的能力，而这个自省的过程就是反射，反射直观的理解就是根据到达地找出出发地和来源

反射指在PHP运行状态中，扩展分析PHP程序，到处或提取出类，方法，属性，参数等详细信息，包括注释

<!--more-->

### 示例

```php

<?php

class Person {
    public $name;
    public $gender;

    public function say()
    {
        echo $this->name ':' $this->gender;
    }

    public function __set($name, $value)
    {
        echo 'setting $name to $value';

        $this->$name = $value;
    }

    public function __get($name)
    {
        if (!isset($this->$name)) {
            echo '未设置';
            $this->$name = '默认值';

        }

        return $this->$name;
    }
}
```

```
$student = new Person();

$student->name = 'Ton';
$student->gender = 'male';
$student->age = '24';
```

### 使用API获取属性和方法

```php
//获取对象属性列表

$reflect = new ReflectionObject($student);
$props = $reflect->getProperties();

foreach ($props as $prop) {
    print $prop->getName();
}

//获取对象方法列表
$m = $reflect->getMethods();
foreach ($m as $prop) {
    print $prop->getName();
}
```

### 使用class函数获取属性和方法

```php
//返回对象属性的关联数组
var_dump(get_object_vars($student));
//类属性
var_dump(get_class_vars(get_class($student)));
//返回由类的方法组成的数组
var_dump(get_class_methonds(get_class($student)));
```

### 获取对象属于哪个类

```php
echo get_class($student);
```


### 还原类原型

```php
//反射获取类的原型
$obj = new ReflectionClass('person');
$className = $obj->getName();
$methods = $properties = [];

foreach ($obj->getProperties() as $v) {
    $properties[$v->getName()] = $v;
}

foreach ($obj->getMethods() as $v) {
    $methods[$v->getName()] = $v;
}

echo "class {$className}";

is_array($properties) && ksort($properties);

foreach($properties as $k => $v) {
    echo $v->isPublic ? 'public' : '';
    echo $v->isPrivate ? 'private' : '';
    echo $v->isProtected ? 'protected' : '';
    echo $v->static ? 'static' : '';
}

if (is_array($methods)) ksort($methods);

foreach ($methods as $k => $v) {
    echo "function{$k}(){}";
}

```

PHP手册关于反射API有几十个，反射完整的描述了一个类或对象的原型，反射不仅可以用于类和对象，还可以用于函数，扩展模块，异常等

### 结语

善用反射能保持代码的优雅和简洁，但反射也会破坏类的封装性，因为反射可以使本不该暴露的方法或属性被强制暴露了出来，是优点也是缺点