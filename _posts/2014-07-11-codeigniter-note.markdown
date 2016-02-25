---
layout: post
title:  "CodeIgniter学习笔记"
date:   2014-07-11 23:35:00 +0800
categories: PHP
tags: CodeIgniter PHP
---

## 一. CI介绍

### 1.1 关于CI

CI全称CodeIgniter。最早是由Rick Ellis开发（此人现在是EllisLab 公司的CEO）。这个框架的灵感来自RoR，刚开始是为真实应用而编写的，集成了很多类库、辅助函数(helpers)以及从 ExpressionEngine借用的子系统。而目前CI由“ExpressionEngine开发团队”开发并维护。

### 1.2 服务器要求

PHP版本要求5.2.4及以上版本。 数据库可以支持MySQL、MySQLi、MS SQL Server、Postgres、Oracle和SQLite，另外也支持ODBC。

### 1.3 许可协议

只要符合以下条件，你将被允许使用、复制、修改以及分发本软件和它相关的文档，包括你可以修改或者不修改地用于任何目的：

1) 这个许可协议的一份拷贝必须包含在分发的软件中。  
2) 再分发源代码时必须在所有源代码文件中保留上方的版权提醒。  
3) 以二进制形式再分发时，必须在文档以及/或者随分发提供的其他物品上保留上面的版权提醒。  
4) 任何修改过的文件必须加上对原始代码修改的注释以及修改者名称。  
5) 任何由本软件衍生的产品必须在它们的文档以及/或者随分发提供的物品中表明它们来源于CodeIgniter。  
6) 除非事先得到EllisLab公司的书面许可，否则任何本软件的衍生产品都不得叫做“CodeIgniter”，也不得在名称中出现“CodeIgniter”。

*注意：  
也就是说，使用了CI的产品，在License中必须保留CI提供的License，必须说明该产品基于CI。如果修改了CI的源码，那么必须添加修改注释并签上大名，而且新产品不能在名字中带有"CodeIgniter"字样。*

*更新：
这是2.x以前的，现在已经换成更加自由的MIT协议*

### 1.4 CI的特点

* CodeIgniter是一个应用程序框架  
* CodeIgniter是免费的  
* CodeIgniter是轻量级的  
* CodeIgniter是快速的  
* CodeIgniter使用 M-V-C 模型  
* CodeIgniter生成干净的 URL  
* CodeIgniter功能强大  
* CodeIgniter是可扩展的  
* CodeIgniter不需要模板引擎  
* CodeIgniter已彻底文档化  
* CodeIgniter拥有一个友好的用户社区  

### 1.5 CI的主要功能

CodeIgniter的主要特性如下：

* 基于MVC体系  
* 超轻量级  
* 对数种数据库平台的全特性支持的数据库类  
* Active Record支持  
* 表单与数据验证  
* 安全性与XSS过滤  
* Session管理  
* 邮件发送类，支持附件，HTML 或文本邮件，多协议(sendmail,SMTP和Mail)及更多。  
* 图像处理类库(剪裁，缩放，旋转等)，支持GD，ImageMagick和BetPBM  
* 文件上传类  
* FTP类  
* 本地化  
* 分页  
* 数据加密  
* 基准测试  
* 全页面缓存  
* 错误日志  
* 应用程序评测  
* 日历类  
* User-Agent类  
* Zip 编码类  
* 模板引擎类  
* Trackback类  
* XML-RPC类库  
* 单元测试类  
* “搜索引擎友好”的URL  
* 灵活的URI路由  
* 支持钩子和类扩展  
* 大量的辅助函数  

## 二. CI使用入门

### 2.1 安装与配置

1) 解压安装包  

2) 上传文件  

3) 修改配置文件application/config/config.php  

>这一步将配置网站的根URL，以及加密、Session等。  

4) 修改配置文件application/config/database.php

>这一步将配置数据库参数。  
>此后，可以隐藏CI的文件位置(system和application目录的路径修改后要同步修改index.php中的$systempath和$applicationfolder变量，最好用绝对路径)。

在生产环境，要把index.php的ENVIRONMENT 常量设为'production'，开发时则可以使用'development'。

### 2.2 应用程序流程

1) index.php 作为前端控制器，初始化运行 CodeIgniter 所需要的基本资源。

2) Router 检查 HTTP 请求，以确定谁来处理请求。

3) 如果缓存(Cache)文件存在，它将绕过通常的系统执行顺序，被直接发送给浏览器。

4) 安全(Security)。应用程序控制器(Application Controller)装载之前，HTTP 请求和任何用户提交的数据将被过滤。

5) 控制器(Controller)装载模型、核心库、辅助函数，以及任何处理特定请求所需的其它资源。

6) 最终视图(View)渲染发送到 Web 浏览器中的内容。如果开启缓存(Caching)，视图首先被缓存，所以将可用于以后的请求。

### 2.3 MVC模式

CodeIgniter 是基于模型-视图-控制器这一设计模式的。MVC 是一种将应用程序的逻辑层和表现层进行分离的方法。在实践中，由于表现层从 PHP 脚本中分离了出来，所以它允许你的网页中只包含很少的脚本。

模型(Model)代表你的数据结构。通常来说，你的模型类将包含取出、插入、更新你的数据库资料这些功能。

视图(View)是展示给用户的信息。一个视图通常是一个网页，但是在CodeIgniter中，一个视图也可以是一个页面片段，如页头、页尾。它还可以是一个RSS页面，或任何其它类型的“页面”。

控制器(Controller)是模型、视图以及其他任何处理HTTP请求所必须的资源之间的中介，并生成网页。

CodeIgniter在MVC使用上非常宽松，因此模型不是必需的。如果你不需要使用这种分离方式，或是发觉维护模型比你想象中的复杂很多，你可以不用理会它们而创建自己的应用程序，并最少化使用控制器和视图。CodeIgniter也可以和你现有的脚本合并使用，或者允许自行开发此系统的核心库，可以使你以最适合你的方式工作。

### 2.4 设计和架构目标

CodeIgniter的目标是在最小化，最轻量级的开发包中得到最大的执行效率、功能和灵活性。

为了达到这个目标，我们在开发过程的每一步都致力于基准测试、重构和简化工作，拒绝加入任何对实现目标没有帮助的东西。

从技术和架构角度看，CodeIgniter 按照下列目标创建：

* 动态实例化  
在CodeIgniter中，组件的导入和函数的执行只有在被要求的时候才执行，而不是在全局范围。除了最小的核心资源外，不假设系统需要任何资源，因此缺省的系统非常轻量级。被HTTP请求所触发的事件，以及你设计的控制器和视图将决定它们什么时候被引用。  

* 松耦合
耦合是指一个系统的组件之间的相关程度。越少的组件相互依赖那么这个系统的重用性和灵活性就越好。我们的目标是一个非常松耦合的系统。  

* 组件专一性
专一是指组件有一个非常小的专注目标。在CodeIgniter里，为了达到最大的用途，每个类和它的功能都是高度自治的。  

CodeIgniter是一个动态实例化，高度组件专一性的松耦合系统。它在小巧的基础上力求做到简单、灵活和高性能。

## 三. 加载内容

### 3.1 静态加载

#### 3.1.1 创建中心控制器

在`application/controllers/`下创建一个新的类，名字就是控制器的名称，随便起，要继承于`CI_Controller`类。

#### 3.1.2 创建物理视图（页头、中间、页尾）

在`application/views/`下创建物理视图，页头和页尾是公共的，中间内容因业务而不同。扩展名为php，不要用什么.class.php。

#### 3.1.3 为控制器添加逻辑结构

定义控制器类`Pages`的`view($page)`方法（其实名字无所谓），访问`index.php/pages/view`时就会调用`Pages::view($page)`。

view方法应该实现的，是检查物理视图是否存在，然后使用load方法加载视图，渲染输出。

#### 3.1.4 设置路由

修改路由配置文件`application/config/routes.php`，修改默认`index.php`指向的虚拟视图。

### 3.2 动态加载

#### 3.2.1 建立模型

根据数据库中指定的表建立对应的模型，建立在`application/models`目录下。模型要继承`CI_Model`类，在模型类的构造方法中，要连接数据库：

    $this->load->database();

这样，就可以在模型类中使用`$this->db`访问数据库了。 模型中应该实现对数据的增删改查操作。

#### 3.2.2 建立控制器

对于不同的业务，创建不同的Contorller。这里的控制器，其实有点像JavaEE中的Struts框架做的事情，就是管理虚拟视图应该输出的物理视图。

#### 3.2.3 创建视图

即使是动态加载，也有个模板，模板其实就是静态内容挖几个坑，等待模板引擎把该填的填进去。

#### 3.2.4 设置路由

路由的设置与上一章一样，不过有几点新增的，就是请求可以用"/"分为多级，不仅仅是只有`default_controller`和`(:any)`而已。

## 四. 常规基础话题

### 4.1 对搜索引擎友好的URL

默认情况下，CI的URL有以下形式： `domain.name/index.php/controller/function/param`

`controller`段表示调用的控制器；

`function`表示调用的对应控制器中的函数或方法；

`param`表示传递的参数。

#### 4.1.1 去掉index.php

而index.php这一节是可以去掉的，服务器是Apache的话就修改.htaccess，加上这样的一段：

    RewriteEngine on
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond $1 !^(index\.php|images|robots\.txt)
    RewriteRule ^(.*)$ /index.php/$1 [L]

注意：如果项目不在根目录，最后一句要稍微修改：

    RewriteRule ^(.*)$ index.php/$1 [L]

#### 4.1.2 添加URL后缀

param后面可以加上.html之类的（伪装成静态页面甚至是asp、jsp这样的其他平台……）。

方法其实很简单，只需要在config.php中把`$config['url_suffix']`设置为你想要的后缀就可以了。

#### 4.1.3 传统的查询字符串形式

一开始展示的那个url，也可以表示成`index.php?c=controller&m=function&id=param`

只要在config.php中将`$config['enablequerystrings']`设置为TRUE，然后设置控制器名、方法名的属性名称。

### 4.2 深入了解控制器

#### 4.2.1 控制器类

控制器类要继承于`CI_Controller`类，然后类名必须以大写字母开头。

#### 4.2.2 方法

假如有一个请求URL是：`domain.name/index.php/ctrller/`  
那么安装4.2.1的说法，他会去控制器目录(application/controllers)下载入Ctrller这个控制器类（当然，找不到就报404错误了）。

不过这个URL中没有说明方法名，这个没关系，它会去找默认方法`index`（如果index方法没有定义，那么也是404）。

如果请求URL是这样的：`domain.name/index.php/ctrller/method`  
那么就指定了加载名为method的方法。

#### 4.2.3 参数

还是上面那个URL，后面又多了几节：`domain.name/index.php/ctrller/method/p1/p2/p3/p4`

method/后面的，就属于method方法的参数。调用的方法就是：

    Ctrller::method($p1, $p2, $p3, $p4)

#### 4.2.4 在路由中设置默认控制器

如果我们只访问`domain.name/index.php`，甚至是`domain.name`，不加控制器名称，CI框架就会从路由中寻找默认路由。假设`application/config/routes.php`中设置了

    $route['default_controller'] = 'Ctrller';

那么就相当于访问`domain.name/index.php/ctrller`

#### 4.2.5 调用规则重定义

之前都是在说默认的调用函数的规则，假设我们希望访问`domain.name/index.php/ctrller/method`
时不去调用method，而是其他方法，原先的就做不到了，因为不符合规则。其实可以使用remap方法重新定义这些规则，一下是一个remap示例：

    public function _remap($method, $params = array())
    {
        $method = 'process_'.$method;
        if (method_exists($this, $method))
        {
            return call_user_func_array(array($this, $method), $params);
        }
        show_404();
    }

上面重定义的规则，就是说希望调用对应的以process开头的那些函数，比如传入的方法段是"method"，就去调用processmethod方法，没有的话就404。

#### 4.2.6 处理输出

可以在控制器中定义一个叫"output"的方法，这个方法总是会被调用，因此如果希望总是输出某些内容，就可以写在output()方法中。

#### 4.2.7 禁止访问的方法

如果希望某些方法禁止被访问，除了重定义规则外，还有个选择，可以在方法名称前加上"_"，比如`domain.name/index.php/ctrller/_method`
这个URL就是无效的。

#### 4.2.8 子目录中的控制器

控制器多了，也可以建立子目录进行分类。 比如在application/controllers目录下增加一个论坛相关的控制器的子目录forum，那么访问的URL就应该是类似：
`domain.name/index.php/forum/topic`

### 4.3 深入了解视图(默认模板引擎)

#### 4.3.1 创建视图

物理视图就是一个网页，丢到`application/views/`及其子目录下，逻辑视图就是控制器的方法。以下视图指的是物理视图。

#### 4.3.2 加载视图

只要在逻辑视图的实现中加上

    $this->load->view('name');

就说明加载一个名为name的视图，而这个物理视图对应的文件，就是`application/views/name.php`。

视图可以加载多个，按顺序输出：

    $this->load->view('header');
    $this->load->view('content');
    $this->load->view('footer');

这样就先后加载了页头、内容和页尾。

#### 4.3.3 加载子目录中的视图

物理视图也可以用文件夹归类存放，比如说放在有个视图放在
`application/views/templates/header.php`
加载时语句如下：

    $this->load->view('templates/header');

#### 4.3.4 添加动态数据

加载视图的方法有三个参数，原型大概如下：

    mixed CI_Controller::load::view(
        string $viewname,
        array $data = array(),
        bool $output = false
    );

这里说的是第二个参数。比如要在公共网页头部中使用不同的标题(title)和编码(encoding)，就要传入两个以上参数，使用一个数组就搞定了：

    $data = array(
        'title'=$title, 
        'encoding'=$encoding
    );

然后要在加载视图时把参数传入到视图：

    $this->load->view('templates/header', $data);

需要注意，视图中留给显示动态内容的坑，留的不是`$data['title']`之类的，应该是直接用`$title`。

#### 4.3.5 模板循环

模板语言其实就是简单的php替代语法：  
简单的案例如下，$list代表一个集合，简单来说就是数组，$item是当前循环中用到的集合成员：

    <?php foreach ($list as $item):?>

重复输出内容的HTML形式：

    <?php endforeach;?>

#### 4.3.6 获取视图内容

view的第三个方法可以获取到要加载的视图的内容，以下方法加载myfile视图，并把内容返回到`$output`：

    $output = $this->load->view(
        'myfile', 
        array(), 
        true
    );

如果第三个参数为false，那么只返回视图加载结果，不返回内容。

### 4.4 深入了解模型

这里的模型基本上就是DAO(数据访问对象)，专门用来与数据库打交道。模型类文件应该放在`application/models/`及其子目录下。模型是一个类，必须继承与`CI_Model`类，跟控制器一样，首字母必须大写。

#### 4.4.1 在控制器中使用模型

一般会在控制器的构造器重加载模型，例如有个`application/models/modelname.php`中定义了叫作Modelname模型类，应该这样加载：

    $this->load->model('model_name');

如果这个模型放在forumdao目录下，那么就加上目录名：

    $this->load->model('forumdao/model_name');

加载完成后，就可以通过

    $this->model_name->method()

调用模型的方法。

#### 4.4.2 自动装载模型

可以在`application/config/autoload.php`文件中修改`$autoload['model']`数组的值列表，添加要自动装载的一个或者多个模型。当然自动装载的模型越多，内存开销就越大，只有少数几个类使用到的模型，就不要自动装载了。

#### 4.4.3 连接到数据库

默认情况下，载入一个模型是不会自动连接数据库的，可以在模型的构造方法中连接，也可以设置`Controller::load::model`方法的第三个参数。第三个参数设置为`TRUE`时，装载时自动连接数据库；设置为一个数组时，会用数组中的配置项连接数据库（当需要连接到第二个数据库时可以用）。

### 4.5 关于错误处理

程序难免你会有错误。在CI中，可以建立自己的错误报告，有三种方式：显示错误消息、显示404页面、记录日志文件。

#### 4.5.1 show_error

该函数的原型：

    show_error(
        string $msg, //错误消息
        int $status_code = 500, //状态码
        string $heading = 'An Error Was Encountered' //标题
    );

#### 4.5.2 show_404

该函数的原型：

    show_404(
        string $page, //错误页面
        string $log_error //级别
    );

#### 4.5.3 log_message

该函数的原型：

    log_message(
        string $level, //级别
        string $msg //输出消息
    );

日志有三个级别：debug、info、error

### 4.6 编码规范

#### 4.6.1 文件格式

文件统一使用无BOM的UTF-8编码保存，使用UNIX的档案格式（以'\n'作为行结束符）。

#### 4.6.2 PHP闭合标签

文件结束时忽略所有?>，但是要添加如下格式的文件结束注释：

    /* End of file 文件名 */
    /* Location: ./文件路径 */

#### 4.6.3 类、方法、函数、变量的命名规范

类的首字母都应该大写，如果有多个词构成，词之间以"_"分隔，不要使用骆驼命名法(就是ClassName这样的)。

类中的方法名以及函数名应该全部小写，并且明确指出用途，最好以动词开头，但不要过长。

变量名与函数名类似，只能使用小写字母，短的、无意义的变量名只应该作为迭代器存在于for循环中。

#### 4.6.4 注释

块注释只用在DOC和文件结束；多行的行注释与代码之间要有空行。

#### 4.6.5 常量

常量的名称全部大写，其他规则同变量。`TRUE`、`FALSE`和`NULL`这三个变量也一样。

#### 4.6.6 一个类一个文件

定义类的时候，应该只在一个文件中定义一个类。

#### 4.6.7 缩进规则

使用空格代替tab；括号使用C++的风格，即前括号应该单独成行；不要在[]和()中间加空格。

#### 4.6.8 私有方法和变量

名字前要加上下划线(_)。

#### 4.6.9 每行一条语句

不要在一行中写多条语句。

#### 4.6.10 字符串

在不解析变量时，用单引号；在解析变量时，用双引号，但变量要加上{}，引号嵌套时，使用双引号包含单引号。

#### 4.6.11 SQL

关键字全部大写，每行只写一个从句。

#### 4.6.12 默认参数

应该适当提供默认参数。
