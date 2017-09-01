# 你所不知道的 Resource

## 前言
很久没写文章了，在新的公司新的遇到了新的伙伴，胖丁哥哥让我看了 `laracon 2017 - Adam Wathan` 的视频，略微手痒想分享一下自己的东西，这边使用的是 `laravel` 作为讲解，但是思想却不局限于于 `laravel` 或者 `php`。
呦呦呦呦，这边是差不多的小二先生~~~~

## 差不多的 路由
平时我们写代码的时候经常会写出很多下面这样的路由：
```
Route::get('/podcasts', 'PodcastsController@index');
Route::get('/podcasts/create', 'PodcastsController@create');
Route::post('/podcasts', 'PodcastsController@store');
Route::get('/podcasts/{id}', 'PodcastsController@show');
Route::get('/podcasts/{id}/edit', 'PodcastsController@edit');
Route::patch('/podcasts/{id}', 'PodcastsController@update');
Route::delete('/podcasts/{id}', 'PodcastsController@destroy');

Route::post('/podcasts/{id}/update-cover-image', 'PodcastsController@updateCoverImage');
Route::post('/podcasts/{id}/subscribe', 'PodcastsController@subscribe');
Route::post('/podcasts/{id}/unsubscribe', 'PodcastsController@unsubscribe');
Route::post('/podcasts/{id}/publish', 'PodcastsController@publish');
Route::post('/podcasts/{id}/unpublish', 'PodcastsController@unpublish');

Route::get('/episodes', 'EpisodesController@index');
Route::get('/episodes/{id}', 'EpisodesController@show');
Route::get('/episodes/{id}/edit', 'EpisodesController@edit');
Route::patch('/episodes/{id}', 'EpisodesController@update');

Route::get('/podcasts/{id}/episodes', 'PodcastController@indexEpisode');
Route::post('/podcasts/{id}/episodes', 'PodcastController@storeEpisode');
Route::get('/podcasts/{id}/episodes/new', 'PodcastController@createEpisode');
```
应该非常熟悉这样所谓 `嵌套资源`，随着项目的扩大，这样会使得控制器一个个的变得胖起来逻辑开始复杂起来，现在让我们开始为这差不多的路由做个变身。

## 差不多的 CURD / REST
这边进行一个小插曲，对资源总结起来大概就是 7 个标准的 Action ：
* Index - 用来展示所有的资源项，比如所有用户。
* Show - 用来展示单个的资源项，比如用户详情。
* Create - 用来显示创建资源的页面，前后端分离可能就没这个 Action 。
* Store - 用来接受数据并创建资源项，比如创建用户。
* Edit - 用来显示编辑资源的页面，前后端分离可能就没这个 Action 。
* Update - 用来接受数据并修改资源项，比如保存用户详情。
* Destroy - 用来删除指定的资源项，比如删除用户。

在后面的路由列表中，我们把只带有这 7 种 Action 的路由器都写成 `Resource`。

## 差不多的 小问题
在上面的路由中我们选择一条常见的路由来做变身：
```
GET /podcasts/{id}/episodes
```
对于这样的一个 URL，如果我们只想让控制器只拥有 7 个标准的 Action ，我们应该把它放在哪个控制器呢？

### 差不多的 控制器
放在 PodcastsController 控制器中吗？那这样就会与控制器中的 `Index`  Action 冲突了。
放在 EpisodesController 控制器中吗？这样也会与控制器中的 `Index`  Action 冲突。

```
GET /podcasts/{id}/episodes => Index
GET /podcasts/              => Index
GET /episodes/              => Index
```

那我们需要怎么安放这个到处被人嫌弃的 URL 呢？

### 不一样的 控制器
其实我们可以把 podcasts 和 episodes 合起来当做一种资源，存放在 `PodcastEpisodesController` 中。
```
class PodcastEpisodesController extends Controller
{
    public function index($id)
    {
        $podcast = Podcast::with('episodes')->findOrFail($id);
        return response()->json(['podcast' => $podcast]);
    }
}
```

### 不一样的 路由
```
Route::resource('/podcasts', 'PodcastsController');
Route::resource('/episodes', 'EpisodesController');
Route::resource('/podcasts/{id}/episodes', 'PodcastEpisodesController');

Route::post('/podcasts/{id}/update-cover-image', 'PodcastsController@updateCoverImage');
Route::post('/podcasts/{id}/subscribe', 'PodcastsController@subscribe');
Route::post('/podcasts/{id}/unsubscribe', 'PodcastsController@unsubscribe');
Route::post('/podcasts/{id}/publish', 'PodcastsController@publish');
Route::post('/podcasts/{id}/unpublish', 'PodcastsController@unpublish');
```

按照这个思路来进行路由的变身，我们将会得到三个控制器：
* `PodcastsController` 拥有 7个标准 Action，5个非标准的 Action
* `EpisodesController` 拥有 4个标准 Action
* `PodcastEpisodesController` 拥有 3个标准 Action

## 差不多的 问题
虽然经历瘦身后，路由列表已经变得很短了，但是`PodcastsController` 中还有 5 个非标准的 Action，我们将继续对这些方法进行瘦身：
```
POST /podcasts/{id}/update-cover-image
```

### 不一样的 控制器
这个 URL 是用来更新 podcasts 的封面图片的，我们是不能把封面图片也单独看成一种资源呢？显然是可以的，这个资源中包含了一个更新的方法。

```
class PodcastCoverImageController extends Controller
{
    public function update($id)
    {
        $podcast = Auth::user()->podcasts()->findOrFail($id);
        $podcast->update([
            'cover_path' => request()->file('cover_image')->store('images', 'public')
        ]);
        return response()->json(['message' => 'ok']);
    }
}
```
### 不一样的 路由
这个时候新的路由可以写成：
```
Route::put('/podcasts/{id}/cover-image', 'PodcastCoverImageController@update');
```
新的路由表可以写为：
```
Route::resource('/podcasts', 'PodcastsController');
Route::resource('/episodes', 'EpisodesController');
Route::resource('/podcasts/{id}/episodes', 'PodcastEpisodesController');
Route::resource('/podcasts/{id}/cover-image', 'PodcastCoverImageController');

Route::post('/podcasts/{id}/subscribe', 'PodcastsController@subscribe');
Route::post('/podcasts/{id}/unsubscribe', 'PodcastsController@unsubscribe');
Route::post('/podcasts/{id}/publish', 'PodcastsController@publish');
Route::post('/podcasts/{id}/unpublish', 'PodcastsController@unpublish');
```

## 差不多的 中间表问题
刚才我们讨论的两个问题都是对于普通的表进行操作，但是如果我们修改和创建的数据在中间表上我们又该如何呢？

```
Route::post('/podcasts/{id}/subscribe', 'PodcastsController@subscribe');
Route::post('/podcasts/{id}/unsubscribe', 'PodcastsController@unsubscribe');
```

这两个路由，分别对 `user_podcast` 中间表的进行删除数据和创建数据。

### 不一样的 控制器
其实我们可以把中间表也看做一种资源，写成 `SubscriptionsController`，其中里面包含两个 Action 有 `store` 和 `destroy`。按照这个思路可把剩下的两个控制器写入 `PublishedPodcastsController`，也是包含了 `store` 和 `destroy` Action。

### 不一样的 路由
经过这么瘦身下来，我们的路由表变成这个样子：
```
Route::resource('podcasts', 'PodcastsController');
Route::resource('episodes', 'EpisodesController');
Route::resource('podcasts.episodes', 'SubscriptionsController');
Route::resource('podcasts.cover-image', 'PodcastCoverImageController');
Route::resource('subscriptions', 'SubscriptionsController');
Route::resource('published-podcasts', 'PublishedPodcastsController');
```
惊喜不惊喜，刺激不刺激，好看不好看，简洁不简洁！！！

## 结尾
其实，我们可以把 `Everything` 都看做是资源，对其进行 `CURD` 的操作，带来的好处也是显而易见，更加轻的控制器，更加进行的分类，更加的 RESTful。
