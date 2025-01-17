# Nano, by Hyperf

Nano 是一款零配置、無骨架、極小化的 Hyperf 發行版，通過 Nano 可以讓您僅僅通過 1 個 PHP 檔案即可快速搭建一個 Hyperf 應用。   

## 設計理念

`Svelte` 的作者提出過一個論斷：“框架不是用來組織程式碼的，是用來組織思路的”。而 Nano 最突出的一個優點就是不打斷你的思路。Nano 非常擅長於自我宣告，幾乎不需要了解框架細節，只需要簡單讀一讀程式碼，就能知道程式碼的目的。通過極簡的程式碼宣告，完成一個完整的 Hyperf 應用。

## 特性

* 無骨架
* 零配置
* 快速啟動
* 閉包風格
* 支援註解外的全部 Hyperf 功能
* 相容全部 Hyperf 元件
* Phar 友好

## 安裝

```bash
composer require hyperf/nano
```

## 快速開始

建立一個 PHP 檔案，如 index.php 如下：

```php
<?php
use Hyperf\Nano\Factory\AppFactory;

require_once __DIR__ . '/vendor/autoload.php';

$app = AppFactory::create();

$app->get('/', function () {

    $user = $this->request->input('user', 'nano');
    $method = $this->request->getMethod();

    return [
        'message' => "hello {$user}",
        'method' => $method,
    ];

});

$app->run();
```

啟動服務：

```bash
php index.php start
```

簡潔如此。

## 更多示例

### 路由

`$app` 集成了 Hyperf 路由器的所有方法。

```php
<?php
use Hyperf\Nano\Factory\AppFactory;

require_once __DIR__ . '/vendor/autoload.php';

$app = AppFactory::create();

$app->addGroup('/nano', function () use ($app) {
    $app->addRoute(['GET', 'POST'], '/{id:\d+}', function($id) {
        return '/nano/'.$id;
    });
    $app->put('/{name:.+}', function($name) {
        return '/nano/'.$name;
    });
});

$app->run();
```

### DI 容器
```php
<?php
use Hyperf\Nano\ContainerProxy;
use Hyperf\Nano\Factory\AppFactory;

require_once __DIR__ . '/vendor/autoload.php';

class Foo {
    public function bar() {
        return 'bar';
    }   
}

$app = AppFactory::create();
$app->getContainer()->set(Foo::class, new Foo());

$app->get('/', function () {
    /** @var ContainerProxy $this */
    $foo = $this->get(Foo::class);
    return $foo->bar();
});

$app->run();
```
> 所有 $app 管理的閉包回撥中，$this 都被繫結到了 `Hyperf\Nano\ContainerProxy` 上。

### 中介軟體
```php
<?php
use Hyperf\Nano\Factory\AppFactory;

require_once __DIR__ . '/vendor/autoload.php';

$app = AppFactory::create();

$app->get('/', function () {
    return $this->request->getAttribute('key');
});

$app->addMiddleware(function ($request, $handler) {
    $request = $request->withAttribute('key', 'value');
    return $handler->handle($request);
});

$app->run();
```

> 除了閉包之外，所有 $app->addXXX() 方法還接受類名作為引數。可以傳入對應的 Hyperf 類。

### 異常處理

```php
<?php
use Hyperf\HttpMessage\Stream\SwooleStream;
use Hyperf\Nano\Factory\AppFactory;

require_once __DIR__ . '/vendor/autoload.php';

$app = AppFactory::create();

$app->get('/', function () {
    throw new \Exception();
});

$app->addExceptionHandler(function ($throwable, $response) {
    return $response->withStatus('418')
        ->withBody(new SwooleStream('I\'m a teapot'));
});

$app->run();
```

### 命令列

```php
<?php
use Hyperf\Contract\StdoutLoggerInterface;
use Hyperf\Nano\Factory\AppFactory;

require_once __DIR__ . '/vendor/autoload.php';

$app = AppFactory::create();

$app->addCommand('echo', function(){
    $this->output->info('A new command called echo!');
});

$app->run();
```

執行

```bash
php index.php echo
```

### 事件監聽

```php
<?php
use Hyperf\Contract\StdoutLoggerInterface;
use Hyperf\Framework\Event\BootApplication;
use Hyperf\Nano\Factory\AppFactory;

require_once __DIR__ . '/vendor/autoload.php';

$app = AppFactory::create();

$app->addListener(BootApplication::class, function($event){
    $this->get(StdoutLoggerInterface::class)->info('App started');
});

$app->run();
```

### 自定義程序

```php
<?php
use Hyperf\Contract\StdoutLoggerInterface;
use Hyperf\Nano\Factory\AppFactory;

require_once __DIR__ . '/vendor/autoload.php';

$app = AppFactory::create();

$app->addProcess(function(){
    while (true) {
        sleep(1);
        $this->get(StdoutLoggerInterface::class)->info('Processing...');
    }
});

$app->run();
```

### 定時任務

```php
<?php
use Hyperf\Contract\StdoutLoggerInterface;
use Hyperf\Nano\Factory\AppFactory;

require_once __DIR__ . '/vendor/autoload.php';

$app = AppFactory::create();

$app->addCrontab('* * * * * *', function(){
    $this->get(StdoutLoggerInterface::class)->info('execute every second!');
});

$app->run();
```

### 使用更多 Hyperf 元件

```php
<?php
use Hyperf\DB\DB;
use Hyperf\Nano\Factory\AppFactory;

require_once __DIR__ . '/vendor/autoload.php';

$app = AppFactory::create();

$app->config([
    'db.default' => [
        'host' => env('DB_HOST', 'localhost'),
        'port' => env('DB_PORT', 3306),
        'database' => env('DB_DATABASE', 'hyperf'),
        'username' => env('DB_USERNAME', 'root'),
        'password' => env('DB_PASSWORD', ''),
    ]
]);

$app->get('/', function(){
    return DB::query('SELECT * FROM `user` WHERE gender = ?;', [1]);
});

$app->run();
```
