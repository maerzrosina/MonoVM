## 前言

本文将详细介绍如何在项目中集成PayPal的两种主要支付方式：Checkout收银台支付和Subscription订阅计划。通过完整的流程讲解和代码示例，帮助开发者快速实现PayPal支付功能。

## 一、支付流程概述

### 1. Checkout收银台支付流程

Checkout模式类似支付宝的收银台支付，主要步骤如下：

1. 本地应用组装参数请求Checkout接口，获取支付URL
2. 重定向到支付URL，用户登录PayPal并确认支付
3. 支付完成后跳转回本地应用
4. 本地请求PayPal执行付款
5. 处理异步通知，验签后更新订单状态

### 2. Subscription订阅计划流程

订阅模式适用于周期性收费场景，步骤如下：

1. 创建并激活订阅计划
2. 基于激活的计划创建订阅申请
3. 用户授权订阅并完成首次付款
4. 执行订阅
5. 处理订阅回调和支付结果

## 二、技术实现

### 1. 环境准备

安装PayPal SDK：
bash
composer require paypal/rest-api-sdk-php:*


👉 [野卡 | 一分钟注册，轻松订阅海外线上服务](https://bit.ly/bewildcard)

### 2. 配置文件

创建 `config/paypal.php`:
php
<?php
return [
    'sandbox' => [
        'client_id' => env('PAYPAL_SANDBOX_CLIENT_ID', ''),
        'secret' => env('PAYPAL_SANDBOX_SECRET', ''),
        'notify_web_hook_id' => env('PAYPAL_SANDBOX_NOTIFY_WEB_HOOK_ID', ''),
    ],
    'live' => [
        'client_id' => env('PAYPAL_CLIENT_ID', ''),
        'secret' => env('PAYPAL_SECRET', ''),
        'notify_web_hook_id' => env('PAYPAL_NOTIFY_WEB_HOOK_ID', ''),
    ],
];


### 3. 服务类实现

创建PayPal服务类处理核心逻辑：
php
<?php
namespace App\Services;

use PayPal\Rest\ApiContext;
use PayPal\Auth\OAuthTokenCredential;

class PayPalService 
{
    protected $config;
    protected $apiContext;
    
    public function __construct($config)
    {
        $this->config = $config;
        $this->apiContext = new ApiContext(
            new OAuthTokenCredential(
                $this->config['client_id'],
                $this->config['secret']
            )
        );
        
        $this->apiContext->setConfig([
            'mode' => $this->config['mode'],
            'log.LogEnabled' => true,
            'log.FileName' => storage_path('logs/PayPal.log'),
            'log.LogLevel' => 'DEBUG',
            'cache.enabled' => true,
        ]);
    }
}


### 4. 路由配置

在 `routes/web.php` 中添加:
php
// Checkout相关路由
Route::get('payment/{order}/paypal', 'PaymentController@payByPayPalCheckout')
    ->name('payment.paypal_checkout');
Route::get('payment/paypal/return', 'PaymentController@payPalReturn')
    ->name('payment.paypal.return');
Route::post('payment/paypal/notify', 'PaymentController@payPalNotify')
    ->name('payment.paypal.notify');

// Subscription相关路由
Route::get('subscriptions/{order}/paypal/plan', 'SubscriptionsController@createPlan')
    ->name('subscriptions.paypal.createPlan');
Route::get('subscriptions/paypal/return', 'SubscriptionsController@execute')
    ->name('subscriptions.paypal.return');
Route::post('subscriptions/paypal/notify', 'SubscriptionsController@payPalNotify')
    ->name('subscriptions.paypal.notify');


## 三、注意事项

1. PayPal回调地址必须使用HTTPS协议
2. 沙箱环境响应较慢，测试时需要耐心
3. 建议对所有支付操作进行日志记录
4. 务必正确配置Webhook事件
5. 做好异常处理和失败重试机制

## 总结

本文详细介绍了PayPal支付的两种主要模式的实现方法。完整代码示例请参考文章内容，根据实际需求进行调整。如需更多支付解决方案，欢迎使用野卡一站式服务。

👉 [野卡 | 一分钟注册，轻松订阅海外线上服务](https://bit.ly/bewildcard)