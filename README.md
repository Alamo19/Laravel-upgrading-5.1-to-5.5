# Laravel-upgrading-5.1-to-5.5

### Composer镜像加速：
   ```bash
   $> composer config -g repo.packagist composer https://mirrors.aliyun.com/composer
   ``` 

- https://packagist.phpcomposer.com

- https://mirrors.aliyun.com/composer

- https://mirrors.cloud.tencent.com/composer

- https://php.cnpkg.org

- https://mirrors.huaweicloud.com/repository/php

### 版本之间文件修改对比查看地址：

https://github.com/laravel/laravel/compare/5.3...5.4

以下只列出部分修改，其他修改请参照各版本的升级指南

### Laravel5.1升级到5.2
#### 1、更新依赖

   删除`illuminate/html`、`laravelcollective/html`包，使用composer remove 包名命令
   
   更新`composer.json`
    
   Require部分`laravel/framework 5.2.*`项目所有依赖检查是否支持5.2框架版本并更新
   
   Require-dev部分新增`symfony/dom-crawler ~3.0`和`symfony/css-selector ~3.0`

   执行
   ```bash
   $> composer update -vvv
   ```         
   如果出现错误`Something's changed, looking at all rules again (pass #23)`，说明项目依赖包与框架包不匹配
#### 2、config调整
   `app.php`文件中,删除
   
   `Illuminate\Foundation\Providers\ArtisanServiceProvider` 
         
   `Illuminate\Routing\ControllerServiceProvider`
   
   `Illuminate\Html\HtmlServiceProvider`

   `App\Providers\BusServiceProvider`

#### 3、Controller.php基类
     
`use Illuminate\Foundation\Bus\DispatchesCommands`

更换为

`use Illuminate\Foundation\Bus\DispatchesJobs;`

`use DispatchesCommands`

更换为

`use DispatchesJobs`

#### 4、.env配置文件变量更新

    APP_DOMAIN_ROOT=laraveltest.com
    APP_DOMAIN_ADMIN=admin.{$APP_DOMAIN_ROOT}

{$PARAM}变量为${PARAM}

#### 5、Jobs调整
在创建任务/命令时你不再需要实现`SelfHandling`契约

#### 6、database.php中`connections.mysql.strict`
`strict`为`false`时

5.1版本中
`set session sql_mode=''`

5.2及之后版本
`set session sql_mode='NO_ENGINE_SUBSTITUTION'`

可通过配置项`modes`进行兼容

### Laravel5.2升级到5.3
#### 1、更新依赖
更新`composer.json`
 
Require部分`laravel/framework 5.3.*`项目所有依赖检查是否支持5.2框架版本并更新
         
Require-dev部分`phpunit/phpunit  ~5.0 `、`symfony/dom-crawler  3.1.*`、`symfony/css-selector 3.1.*`

执行
   ```bash
   $> composer update -vvv
   ```         
   如果出现错误`Something's changed, looking at all rules again (pass #23)`，说明项目依赖包与框架包不匹配

laravel/tinker安装

#### 2、Providers文件修改
`EventServiceProviders.php`

```php
public function boot(DispatcherContract $events)
    {
        parent::boot($events); 
```

改为

```php
public function boot()
    {
    parent::boot();
``` 


`RouteServiceProvider.php`

```php
public function boot(Router $router)
{
    //
    parent::boot($router);
```

改为

```php
public function boot()
{
    //
    parent::boot(); 
``` 

#### 3、scode错误

`Interface 'Illuminate\Database\Eloquent\ScopeInterface' not found`

使用到的地方调整为`Illuminate\Database\Eloquent\Scope`

#### 4、路由中间件

5.3及以后的版本`controller`的构造方法`____construct`无法获取`session`

原因5.3之前版本路由中间件先执行再实例控制器
`Illuminate\Routing\Router.php`
```php
    protected function runRouteWithinStack(Route $route, Request $request)
    {
        $shouldSkipMiddleware = $this->container->bound('middleware.disable') &&
                                $this->container->make('middleware.disable') === true;

        $middleware = $shouldSkipMiddleware ? [] : $this->gatherRouteMiddlewares($route);

        return (new Pipeline($this->container))
                        ->send($request)
                        ->through($middleware)
                        ->then(function ($request) use ($route) {
                            return $this->prepareResponse(
                                $request,
                                $route->run($request)//此处进行控制器实例化
                            );
                        });
    } 
```
 
5.3及以后版本中获取控制器路由中间件时会先实例化控制器再执行路由中间件
`Illuminate\Routing\Router.php`
```php
    protected function runRouteWithinStack(Route $route, Request $request)
    {
        $shouldSkipMiddleware = $this->container->bound('middleware.disable') &&
                                $this->container->make('middleware.disable') === true;

        $middleware = $shouldSkipMiddleware ? [] : $this->gatherRouteMiddleware($route);//此处获取路由中间件是实例化控制器

        return (new Pipeline($this->container))
                        ->send($request)
                        ->through($middleware)
                        ->then(function ($request) use ($route) {
                            return $this->prepareResponse(
                                $request, $route->run()
                            );
                        });
    }
```

可用方法如下：

```php
    $this->middleware(function ($request, $next) {
            $passport = $request->session()->get('passport');
            
            return $next($request);
        }); 
``` 

或者
`App\Http\Kernel`的`$middleware`属性开启`session`中间件`StartSession`、`EncryptCookies`
```php
    protected $middleware = [
        \Illuminate\Foundation\Http\Middleware\CheckForMaintenanceMode::class,
        \Illuminate\Foundation\Http\Middleware\ValidatePostSize::class,
        \App\Http\Middleware\TrimStrings::class,
        \Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull::class,
        \App\Http\Middleware\TrustProxies::class,
		\App\Http\Middleware\EncryptCookies::class,
        \Illuminate\Session\Middleware\StartSession::class,
    ]; 
``` 

### Laravel5.3升级到5.4


#### 1、更新依赖
更新`composer.json`
 
Require部分`laravel/framework 5.4.*`项目所有依赖检查是否支持5.4框架版本并更新

Require-dev部分`phpunit/phpunit  ~5.7` 
       
执行
   ```bash
   $> composer update -vvv
   ```         
   如果出现错误`Something's changed, looking at all rules again (pass #23)`，说明项目依赖包与框架包不匹配
   

#### 2、Session

`Session::set()`改为`Session::put()`

`$request->session()->set()`改为`$request->session()->put()`


#### 3、UrlGenerator

`Illuminate\Routing\UrlGenerator`类的`forceSchema`方法已经被重命名为`forceScheme`

#### 4、Model

5.4版本开始获取属性时，会调用
`Illuminate\Routing\Database\Eloquent\Concerns\HasAttributes.php`
的`getAttributeValue`方法,自增主键及`$casts`属性中设置的类型会进行强制转换

作用域`Illuminate\Routing\Database\Eloquent\Concerns\HasGlobalScopes.php`

新增事件`Illuminate\Routing\Database\Eloquent\Concerns\HasEvents.php`

#### 5、pagination

`Illuminate\Pagination\AbstractPaginator.php`文件中

`addQuery`方法修饰由`public`改为`protected`

直接调用`addQuery`方法的地方可改为调用`appends`方法


### Laravel5.4升级到5.5

#### 1、更新依赖
更新`composer.json`
 
Require部分`php >=7.0.0 `,`laravel/framework 5.5.*`,`fideloper/proxy:~3.3`项目所有依赖检查是否支持5.5框架版本并更新
         
Require-dev部分`phpunit/phpunit  ~6.0` ,`filp/whoops ~2.0`
       
执行
   ```bash
   $> composer update -vvv
   ```         
   如果出现错误`Something's changed, looking at all rules again (pass #23)`，说明项目依赖包与框架包不匹配
      

#### 2、Config
`app.php`删除`'Laravel\Tinker\TinkerServiceProvider',`

#### 3、index.php

`public/index.php`

```php
require __DIR__.'/../bootstrap/autoload.php';
```

改为

```php
require __DIR__.'/../vendor/autoload.php'; 
``` 

#### 4、cookie中间件EncryptCookies

5.4版本及5.5低版中，`cookie`加密时默认`serialize`序列化，解密时默认`unserialize`反序列化。5.5高版中cookie加解密默认不进行序列化。

`Illuminate\cookie\Middleware\EncryptCookies.php`

低版本中

```php
protected function encrypt(Response $response)
{
        foreach ($response->headers->getCookies() as $cookie) {
            if ($this->isDisabled($cookie->getName())) {
                continue;
            }

            $response->headers->setCookie($this->duplicate(
                $cookie, $this->encrypter->encrypt($cookie->getValue())
            ));
        }

        return $response;
 }
```

高版本中

```php
protected function encrypt(Response $response)
{
        foreach ($response->headers->getCookies() as $cookie) {
            if ($this->isDisabled($cookie->getName())) {
                continue;
            }

            $response->headers->setCookie($this->duplicate(
                $cookie, $this->encrypter->encrypt($cookie->getValue(), static::serialized($cookie->getName()))
            ));
        }

        return $response;
}

```

`$this->encrypter`为`Illuminate\Encryption\Encrypter.php`
方法`encrypt($value, $serialize = true)`

多个系统利用`redis`进行共享`session`时，如果版本不一致导致`cookie`加解密失败，
如果希望跟之前保持一致可修改`app\Http\Middelware\EncryptCookies.php`文件将父类`Illuminate\cookie\Middleware\EncryptCookies`的`$serialize`属性设置为`true`

```php
class EncryptCookies extends Middleware
{

    protected static $serialize = true;

    /**
     * The names of the cookies that should not be encrypted.
     *
     * @var array
     */
    protected $except = [
        //
    ];
}
```
