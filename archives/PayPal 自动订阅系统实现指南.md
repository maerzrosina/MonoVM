**概述：**

PayPal 自动订阅系统的实现涉及多个关键步骤，本文将详细介绍从环境配置到订阅管理的完整实现流程，帮助开发者快速构建可靠的订阅支付系统。

**环境配置：**

首先安装 PayPal PHP SDK：

bash
composer require paypal/rest-api-sdk-php


**实现步骤：**

1. **创建并激活订阅计划**
   - 为不同价格商品创建独立的订阅计划
   - 支持在创建协议时针对用户调整计划内容
   - 创建 `TRIAL` 类型支付时必须配合 `REGULAR` 类型
   - 新用户优惠需要在业务代码中实现

2. **用户订阅流程**
   - 创建订阅协议并获取 PayPal 授权链接
   - 提取 token 用于关联用户授权
   - 处理用户授权后的回调

php
$link = $agreement->getApprovalLink();
parse_str(parse_url($link, PHP_URL_QUERY), $params);
$token = $params['token'];


3. **支付处理要点**
   - 协议生效时间必须在当前时间24小时后
   - 使用 `setSetupFee` 实现首次即时收费
   - 考虑实际扣款延迟，建议提前一天执行
   - 处理重复订阅情况

4. **通知管理**
   - 配置 webhook 接收支付完成通知
   - 使用 `Agreement::searchTransactions` 查询交易
   - 通过 `billing_agreement_id` 匹配订阅协议
   - 处理 `PAYMENT.SALE.COMPLETED` 事件

**重要注意事项：**

- SDK 可能报 `"NotifyUrl" value is NULL` 错误，参考官方解决方案
- 实际扣款时间会有延迟，需要做好重试机制
- 同一用户可多次订阅同一计划，需要考虑如何处理
- 使用 `AgreementDetail.next` 参数跟踪下次收款时间

👉 [野卡 | 一分钟注册，轻松订阅海外线上服务](https://bit.ly/bewildcard)

**开发调试：**

使用 PayPal Sandbox 环境进行完整功能测试，确保系统稳定可靠后再部署到生产环境。注意处理各种异常情况，确保支付流程的连续性和可靠性。

👉 [野卡 | 一分钟注册，轻松订阅海外线上服务](https://bit.ly/bewildcard)