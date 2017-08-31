# Laravel 的升级之路

## 时间线
* Why Modern PHP Is Amazing And How You Can Use It Today. [PeersConf 2014]
* Sharing Laravl. [Laracon EU 2014]
* Leveraging Laravel. [Laracon US 2015]
* Empathy Gives You Superpowers. [Laracon EU 2015]
* Mastering the Illuminate Container. [Laracon Online 2017]

## 如何开始学习 Laravel
### Laravel 理念
* We've got the best defaults - 主厨精选
* Convention over configuration - 约定大于配置
* Convention and (when helpful) configuration - 约定可定制

### Laravel 可定制范围
* /app/* (内置可修改的代码)
* Container, Illuminate Contracts.
* Macros, facades, re-binding, etc.
* Environment variables and conditional config.

### 如何去探索
* 在任何 app 中最经常被问的是 "Where/How is this happening".
* 最优先的解决方法是阅读 laravel 文档
* 或者是阅读 `laravel up and running` 书籍
* 在探索这个过程中会经历从 `啊嘞？这是什么操作` 到 `脑补出代码的实现`

### 探索注意事项
* 不容易被我们理解的东西并不一定是坏的（例如 Facades）
* 最好用的东西并不一定是最容易理解的（例如 Eloquent）
* 我们需要提升自己的能力

### 可能遇到的问题
* 覆盖默认方法（例如 create）
* 修改了 $request 对象
* 不理解 Route::controller
* 错误的使用 Facade 方法
* 觉得 Laravel 很简单 (例如 Tag::create(request()->all()))
* 其实 Laravel 做了一系列很精妙处理

## 提升 Laravel 能力的提示和技巧
### Container 快速入门
#### 大纲
* 注册实例或者类到 Container
* 解析和获取需要的值 Container
* Container 中的 Laravel
* 自动关联和绑定类
* Servive Providers
* aliases 和 Facades

#### 注册实例或者类到 Container
* 键值对存储：把 类或者实例、闭包 放入 自定的键值中
* Key: 可以是一个类的全名或者是一个简称 (例如`logger`)
* Value: 可以是 类、闭包、对象实例
* `app()->bind('address', Thing::class)`

#### 解析和获取需要的值 Container
* 使用 Key 获取 Value
* `app()->make('address')`
* 可以利用这个实现依赖注入 (例如 Controller)

#### Container 中的 Laravel
* 任何 Laravel 的任何东西都是由 Container 创建
* 例如 Controller, middleware
* 函数自身自动的依赖注入
* `public function cool(APIClient $client)`

#### 自动关联和绑定类
* Container 知道如何创建默认的类
* 单例模式：当类已经被创建过后，就不再创建
* 所有的依赖注入都是使用 Container 完成的
* `app()->bind(Thing::class, () => new App\Thing('dz'))`

#### Aliases 和 Facades
* 给 Value 取一个别名
* 可以直接使用这个别名创建类 new Value()
* Facades 可以创建一个方法并像静态方法那样使用该类

### 101: 在能力范围内做适当的定制
#### 配置
* 定制 config 文件
* 定制 config 中的值
* 使用 .env 来导入值到 config 中
* 禁止直接绕过 config 使用 env()

#### 控制器
* 定制自带的控制器 (AuthController)
* 定制自带的视图 (auth.passwords.email)
* 定制自带的路由 (LoginController@redirectTo)
* 覆盖保护的方法 (validateEmail())
* 覆盖 hook 方法 (LoginController@username())

#### 服务提供者
* AuthServiceProvider@boot 和 EventServiceProvider@boot
* RouteServiceProvider@boot,@map

#### 中间件
* EncryptCookies@except, TrimStrings@except, VerifyCsrfTokens@except
* RedirectIfAuthenticated:redirect('/home')
* App\Http\Kernel@$middlewareGroups

#### 自建中间件
```
class KeepOutBadPeople
{
    public function handle($request, Closure $next)
    {
        if ($this->isBad($request->person)) {
            abort(404);
        }
        return $next($request);
    }
}
```

#### 异常
* 定制 exceptions (e.g. TryingToHackUsWhatTheHeckException)
* 定制全局异常处理 (e.g. App\Exceptions\Handler@report)

#### Include
* 加载和初始化自己需要的代码文件
* 例如 `routes/console.php` 和 `App/Console/Kernel.php`

### 201
#### 测试
* 添加测试断言
* 模拟和绑定需要的类、migrate、seed 在 `setUp()`
* 定制测试配置 `phpunit.xml`
* 定制测试环境 `.env.testing`
* 使用 `migration trait` 在重置数据库
* 使用 `Facade::shouldReceive` 来进行模拟测试

#### 添加帮助函数
```
// 使用 composer 来添加帮助函数
{
    "autoload": {
        "files": ["helpers.php"]
    }
}
```

#### 更进一步的使用 container 来进行绑定
* 绑定类 app()->bind(MyClass::class, Closure)
* 绑定接口 app()->bind(MyInterface::class, ClosureOrClassName)
* 绑定别名 app()->bind('myalias', ClosureOrClassName)
* 创建别名 app()->alias('myalias', SomeContractClassName)

#### Contracts
* 使用所有的 Laravel 接口组件
* Illuminate\Contracts\*

#### 使用包提供的工具
* php artisan vendor:publish

#### 自定义 Facades
* 需要继承 Illuminate\Support\Facades\Facade
* 实现 getFacadeAccessor() 方法，返回值为 Container 注册过的值
* 注册 Facades 到 config/app.php@aliases

#### 自定义 Validation
* `5.5` 之前在服务提供者里定制
* `5.5` 之后的定制方法改变

#### Macros
```
Response::macro('jsend', function ($body, $status = 'success') {
    if ($status == 'error') {
        return Response::json(['status' => $status, 'message' =>  $body]);
    }
    return Response::json(['status' => $status, 'data' => $body]);
});

// In a route
return response()->jsend(Order::all());
```

#### Blade 指令
* 在服务提供者里定制
* `5.5` 之前使用 Blade::directive('name', () => {});
* `5.5` 之后使用 Blade::if('name', () => {});

#### 视图组件
```
// 在服务提供者里定制
// Class-based composer
View::composer(
    'minigraph', 'App\Http\ViewComposers\GraphsComposer'
);
// Closure-based composer
View::composer('minigraph', function ($view) {
    return app('reports')->graph()->mini();
});
```

#### 路由的参数绑定
```
// In a service provider
Route::bind('user', function ($value) {
    return App\User::public()
        ->where('name', $value)
        ->first();
});
// In your binding
Route::get('profile/{user}', function (App\User $user) {
//
});
```

#### Requests 定制
```
// Traditional request injection
public function index(Request $request) {}
// Form Request Validation
public function store(StoreCommentRequest $request) {}
// Form Request
class StoreCommentRequest extends FormRequest {
    public function authorize() { return (bool)rand(0,1); }
    public function rules () { return ['validation rules here']; }
}

class StoreCommentRequest extends FormRequest
{
    public function sanitized()
    {
        return array_merge($this->all(), [
            'body' => $this->sanitizeBody($this->input('body'))
        ]);
    }
    protected function sanitizeBody($body)
    {
        return $body;
    }
}
```

### 301
#### 更高级的 路由绑定
```
// In a service provider
Route::bind('topic', function ($value) {
    return array_get([
        'automotive' => 'I love cars!',
        'pets' => 'I love pets!',
        'fashion' => 'I love clothes!'
    ], $value, 'undefined');
});
// In your routes file
Route::get('list/{topic}', function ($topic) {
//
});

// In a service provider
Route::bind('repository', function ($value) {
    return app('github')->findRepository($value);
});
// In your routes file
Route::get('repositories/{repository}', function ($repo) {
//
});
```

#### 更高级的 Request 定制
```
class MyMagicalRequest extends Request {}

// Type-hint in a route
public function index(MyMagicalRequest $request) {}

// Or even re-bind globally in public/index.php
$response = $kernel->handle(
    // $request = Illuminate\Http\Request::capture()
    $request = App\MyMagicalRequest::capture()
);
```

#### Response 定制
```
// 5.5-pro
class MyMagicalResponse extends Response {}
// Return from a route
public function index()
{
    return MyMagicalResponse::forUser(User::find(1337));
}
// Or macro it
Response::macro('magical', function ($user) {
    return MyMagicalResponse::forUser($user);
});
return response()->magical($user);
```

```
class Thing implements Responsable
{
    public function toResponse()
    {
        return 'HEY THIS IS A USER RESPONSE FOR THING NUMBER ' . $this->id;
    }
}
```

#### 定制 Colletion
```
class ItemCollection extends Collection
{
    public function price()
    {
        return $this->sum('amount') + $this->sumTax();
    }
    public function sumTax()
    {
        return $this->reduce(function ($carry, $item) {
            return $carry + $item->tax();
        }, 0);
    }
}

class Item
{
    public function newCollection(array $models = [])
    {
        return new ItemCollection($models);
    }
}
```

#### 拦截 index.php
```
// 可以定制多个 app 文件
if ($response->getStatusCode() != 404) {
    $response->send();
    $kernel->terminate($request, $response);
    exit;
}
```

## 回顾
* 善于去发现和探究问题
* 101: 在能力范围内做适当的修改
* 201: 进行简单的定制
* 301: 覆盖和定制所有的类
* 定制和使用 Homestead
