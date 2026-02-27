# laravel-audit-flow 使用文档（草稿）

> 包名/命名空间：`WuTongWan\Flow`  
> 服务提供者：`WuTongWan\Flow\FlowServiceProvider`  
> 路由文件：`src/routes.php`（发布到项目 `routes/flow.php`）  
> 视图命名空间：`flow::`（来自 `src/views`）  
> 数据库迁移：来自 `src/migrations`

## 1. 功能概述

`laravel-audit-flow` 提供一套“审批/审核流（flow）”的管理与执行能力（从路由看，至少包含）：

- **流程（Flow）管理**
  - 列表：`GET /flow`、`GET /flow/index`
  - 创建：`GET|POST /flow/createflow`
  - 删除：`POST /flow/delflow`

- **类型（Type）管理**（通常用于给流程分类或绑定业务类型）
  - 列表：`GET /flow/type`
  - 创建：`GET|POST /flow/createtype`
  - 删除：`POST /flow/deltype`

- **节点（Node）管理**（流程节点/审批节点）
  - 列表：`GET /flow/node`
  - 创建：`GET|POST /flow/createnode`

- **节点用户/审批人（User）管理**（为节点配置审批人）
  - 创建：`GET|POST /flow/createuser`
  - 删除：`POST /flow/deluser`

- **单据（Bill）/实例（Instance）相关**
  - 列表：`GET /flow/bill`
  - 查看某单据当前节点/节点信息：`GET /flow/show_node`
  - 关闭单据：`POST /flow/bill_close`
  - 审批记录：`GET /flow/records`

- **创建“单据-流程”关联**
  - `GET /flow/create_relations`

从 ServiceProvider 看，该包还提供：
- 可发布的配置文件 `config/flow.php`
- 可发布的路由文件 `routes/flow.php`（供你的 Laravel 项目加载）
- 视图文件（通过 `flow::xxx` 使用）
- 自带迁移（安装后 `php artisan migrate` 创建审批相关表）

此外，容器绑定了 `flow` 服务，返回 `WuTongWan\Flow\Containers\Interactive`，意味着你也可以通过 `app('flow')`（或 Facade，如果仓库里提供）在代码里驱动审批流。

---

## 2. 环境要求

- Laravel（版本未在 README 中声明；从代码风格推测为 5.x/6.x 风格路由）
- PHP 版本取决于 Laravel 版本与包本身（需看 composer.json 才能确认）

> 建议：你把 `composer.json` 和 `src/config/flow.php` 发我，我能把版本要求写准确。

---

## 3. 安装

### 3.1 Composer 安装

在你的 Laravel 项目中执行：

```bash
composer require zhufeng-danke/laravel-audit-flow
```

> 如果 Packagist 上不是这个包名，请以实际包名为准（需要看该仓库 `composer.json` 的 `name` 字段）。

### 3.2 注册 ServiceProvider（Laravel 5.x）

在 `config/app.php` 中添加：

```php
'providers' => [
    // ...
    WuTongWan\Flow\FlowServiceProvider::class,
],
```

Laravel 支持 package auto-discovery 的情况下可能不需要手动添加（仍需 composer.json 的 `extra.laravel` 才能确认）。

---

## 4. 发布资源（配置 / 路由）

ServiceProvider 中定义了 publishes：

- `src/config/flow.php` -> `config/flow.php`
- `src/routes.php` -> `routes/flow.php`

执行发布（命令可能因 Laravel 版本而异）：

```bash
php artisan vendor:publish --provider="WuTongWan\Flow\FlowServiceProvider"
```

发布完成后，你会在项目中得到：

- `config/flow.php`
- `routes/flow.php`

---

## 5. 加载路由

注意：包内的 `FlowServiceProvider` **注释掉了** `loadRoutesFrom(__DIR__.'/routes.php')`，意味着：

- 包本身不会自动把路由注册到应用里；
- 你需要在自己的项目里**手动加载**发布后的 `routes/flow.php`。

推荐做法（Laravel 5.3+）：

### 5.1 在 `RouteServiceProvider` 中加载

编辑 `app/Providers/RouteServiceProvider.php` 的 `map()`（或自定义 mapXxxRoutes）加入：

```php
public function map()
{
    // ...
    $this->mapFlowRoutes();
}

protected function mapFlowRoutes()
{
    $this->loadRoutesFrom(base_path('routes/flow.php'));
}
```

或在 `routes/web.php` 中直接 require：

```php
require base_path('routes/flow.php');
```

---

## 6. 迁移数据库

包在 `boot()` 里调用了：

- `$this->loadMigrationsFrom(__DIR__ . '/migrations');`

因此只要包被加载，迁移会自动被 Laravel 发现。执行：

```bash
php artisan migrate
```

> 迁移文件具体会创建哪些表、字段、索引，需要读取 `src/migrations/*` 才能在文档里写清楚。

---

## 7. 访问与使用（路由/API）

该包路由前缀是 `flow`：

### 7.1 流程管理

- `GET /flow` 或 `GET /flow/index`  
  流程列表页

- `GET /flow/createflow`  
  创建流程表单页

- `POST /flow/createflow`  
  提交创建流程

- `POST /flow/delflow`  
  删除流程

### 7.2 类型管理

- `GET /flow/type`  
  类型列表

- `GET|POST /flow/createtype`  
  创建类型

- `POST /flow/deltype`  
  删除类型

### 7.3 节点管理

- `GET /flow/node`  
  节点列表

- `GET|POST /flow/createnode`  
  创建节点

### 7.4 节点审批人管理

- `GET|POST /flow/createuser`  
  为节点添加审批人/用户

- `POST /flow/deluser`  
  删除节点审批人/用户

### 7.5 单据/实例与记录

- `GET /flow/bill`  
  单据/实例列表（可能展示每个单据当前走到哪个节点）

- `GET /flow/show_node`  
  查看单据对应节点信息

- `POST /flow/bill_close`  
  关闭单据/终止流程

- `GET /flow/records`  
  审批记录列表

### 7.6 建立单据-流程关系

- `GET /flow/create_relations`  
  创建业务单据与流程的关联关系（一般用于：某类单据绑定某个流程定义）

> 以上接口的具体参数名/返回格式，需要读取 `FlowController` 才能写完整示例。

---

## 8. 在代码中使用（服务容器）

ServiceProvider 注册了：

- `app()->bind('flow', fn() => new WuTongWan\Flow\Containers\Interactive());`

因此你可以在代码中：

```php
$flow = app('flow'); // Interactive 实例
```

接下来如何“发起审批/流转/通过/驳回”等，需要看 `Interactive` 类有哪些方法（目前我无法读取 `src/Containers` 目录内容）。

---

## 9. 视图使用

包通过：

```php
$this->loadViewsFrom(realpath(__DIR__ . '/views'), 'flow');
```

注册了视图命名空间 `flow`。因此在你的项目中可以：

```php
return view('flow::xxx');
```

具体有哪些视图文件（例如 index/create 等）需要读取 `src/views` 目录才能列出清单。

---

## 10. 常见问题（FAQ）

### Q1：为什么访问 `/flow` 404？
A：包没有自动注册路由（`loadRoutesFrom` 被注释），你需要手动加载 `routes/flow.php`（见第 5 节）。

### Q2：发布配置/路由文件后没生效？
A：确保执行了 `vendor:publish`，并且你的项目实际加载了 `routes/flow.php`；配置则需要 `config('flow.xxx')` 才会被读取。

---

## 11. 我还需要你提供的信息（用于补全文档）

为了把本文档升级为“完整可运行版”，请你把以下文件内容贴出来（或允许我读取）：

1. `composer.json`（确认包名、PHP/Laravel 版本、auto-discovery）
2. `src/config/flow.php`（配置项说明）
3. `src/migrations/*`（表结构与字段含义）
4. `src/Http/Controllers/FlowController.php`（各接口参数、校验、返回）
5. `src/Containers/Interactive.php`（编程式 API：发起/流转/记录等）
6. `src/Models/*`（核心模型：Flow、Node、Bill、Record 等）

拿到这些后我可以补上：
- 每个接口的请求示例（curl/postman）
- 数据库表 ER 关系与字段字典
- “创建一个审批流并绑定到业务单据”的端到端示例
- 如何在业务代码中接入（创建单据 -> 发起流程 -> 审批动作 -> 查询记录）
