---
title: "在Slim中使用Eloquent"
date:   2016-07-17 08:30:00
tags:
  - PHP
  - Laravel
  - Eloquent
  - ORM
---

你可以在Slim应用程序中使用诸如[Eloquent](https://laravel.com/docs/5.1/eloquent)这种对象-关系映射（ORM）库，来访问数据库。
<!--more-->

## 将Eloquent添加到你的应用程序

```bash
composer require illuminate/database "~5.1"
```

## 配置Eloquent

在Slim的配置文件（src/settings.php）中添加如下数据库连接相关的数组。

```php
<?php
return [
    'settings' => [
        // Slim Settings
        'determineRouteBeforeAppMiddleware' => false,
        'displayErrorDetails' => true,
        'db' => [
            'driver' => 'mysql',
            'host' => 'localhost',
            'database' => 'database',
            'username' => 'user',
            'password' => 'password',
            'charset'   => 'utf8',
            'collation' => 'utf8_unicode_ci',
            'prefix'    => '',
        ]
    ],
];
```

在你的`src/dependencies.php`或者其他地方，添加数据库相关的服务工厂：

```php
// Service factory for the ORM
$container['db'] = function ($container) {
    $capsule = new \Illuminate\Database\Capsule\Manager;
    $capsule->addConnection($container['settings']['db']);

    $capsule->setAsGlobal();
    $capsule->bootEloquent();

    return $capsule;
};
```

## 向控制器传递表的实例对象

```php
$container[App\WidgetController::class] = function ($c) {
    $view = $c->get('view');
    $logger = $c->get('logger')
    $table = $c->get('db')->table('table_name');
    return new \App\WidgetController($view, $logger, $table);
};
```

## 在控制器中查询表

```php
<?php

namespace App;

use Slim\Views\Twig;
use Psr\Log\LoggerInterface;
use Illuminate\Database\Query\Builder;
use Psr\Http\Message\ServerRequestInterface as Request;
use Psr\Http\Message\ResponseInterface as Response;

class WidgetController
{
    private $view;
    private $logger;
    protected $table;

    public function __construct(
        Twig $view,
        LoggerInterface $logger,
        Builder $table
    ) {
        $this->view = $view;
        $this->logger = $logger;
        $this->table = $table;
    }

    public function __invoke(Request $request, Response $response, $args)
    {
        $widgets = $this->table->get();

        $this->view->render($response, 'app/index.twig', [
            'widgets' => $widgets
        ]);

        return $response;
    }
}
```

### 条件查询

```php
...
$records = $this->table->where('name', 'like', '%foo%')->get();
...
```

### 通过主键id查询数据

```php
...
$record = $this->table->find(1);
...
```

## 更多信息

[Eloquent官方文档](https://laravel.com/docs/5.1/eloquent)
[Eloquent中文文档](http://laravel-china.org/docs/5.1/eloquent)
