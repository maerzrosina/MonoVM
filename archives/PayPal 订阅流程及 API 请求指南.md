## 1. PayPal 后台创建产品及计划

### 正式环境创建订阅计划

[PayPal 商户应用中心](https://www.paypal.com/merchantapps/appcenter/acceptpayments/subscriptions)

### 沙盒环境创建订阅计划

[PayPal 沙盒计划](https://www.sandbox.paypal.com/billing/plans)

进入后可以创建产品及计划（正式环境计划），沙盒计划不会在此处显示。可以根据流程创建产品及计划，也可以通过 API 创建。

## 2. API 创建产品及计划

### API 地址

[PayPal API 文档](https://developer.paypal.com/api/rest/)

### 1. 生成 Token

首先在 PayPal 账号中获取 `clientId` 和 `Secret`，然后获取 `access_token`。

- 沙盒: `https://api.sandbox.paypal.com/v1/oauth2/token`
- 正式: `https://api.paypal.com/v1/oauth2/token`

### 2. 创建产品

bash
curl -v -X POST https://api-m.sandbox.paypal.com/v1/catalogs/products \
-H "Content-Type: application/json" \
-H "Authorization: Bearer Access-Token" \
-H "PayPal-Request-Id: PRODUCT-18062020-001" \
-d '{
  "name": "Video Streaming Service",
  "description": "Video streaming service",
  "type": "SERVICE",
  "category": "SOFTWARE",
  "image_url": "https://example.com/streaming.jpg",
  "home_url": "https://example.com/home"
}'


- `name`: 产品名称
- `description`: 产品说明
- `type`: 产品类型（PHYSICAL, DIGITAL, SERVICE）
- `category`: 产品类别
- `image_url`: 产品 logo
- `home_url`: 产品主站地址

### 3. 创建计划

bash
curl -v -X POST https://api-m.sandbox.paypal.com/v1/billing/plans \
-H "Content-Type: application/json" \
-H "Authorization: Bearer Access-Token" \
-H "PayPal-Request-Id: PLAN-18062019-001" \
-d '{
  "product_id": "PROD-XXCD1234QWER65782",
  "name": "Video Streaming Service Plan",
  "description": "Video Streaming Service basic plan",
  "status": "ACTIVE",
  "billing_cycles": [
    {
      "frequency": {
        "interval_unit": "MONTH",
        "interval_count": 1
      },
      "tenure_type": "REGULAR",
      "sequence": 1,
      "total_cycles": 12,
      "pricing_scheme": {
        "fixed_price": {
          "value": "6",
          "currency_code": "USD"
        }
      }
    }
  ],
  "payment_preferences": {
    "auto_bill_outstanding": true,
    "setup_fee": {
      "value": "6",
      "currency_code": "USD"
    },
    "setup_fee_failure_action": "CONTINUE",
    "payment_failure_threshold": 3
  },
  "taxes": {
    "percentage": "0",
    "inclusive": false
  }
}'


- `product_id`: 产品 ID
- `name`: 计划名称
- `description`: 计划说明
- `status`: 计划状态
- `billing_cycles`: 计费周期
- `payment_preferences`: 付款首选项
- `taxes`: 税

### 5. 创建订阅

bash
curl -v -X POST https://api-m.sandbox.paypal.com/v1/billing/subscriptions \
-H "Content-Type: application/json" \
-H "Authorization: Bearer <Access-Token>" \
-H "PayPal-Request-Id: SUBSCRIPTION-21092019-001" \
-d '{
  "plan_id": "P-5ML4271244454362WXNWU5NQ",
  "start_time": "2022-07-21T00:00:00Z",
  "quantity": "20",
  "shipping_amount": {
    "currency_code": "USD",
    "value": "10.00"
  },
  "application_context": {
    "brand_name": "walmart",
    "locale": "en-US",
    "shipping_preference": "SET_PROVIDED_ADDRESS",
    "user_action": "SUBSCRIBE_NOW",
    "payment_method": {
      "payer_selected": "PAYPAL",
      "payee_preferred": "IMMEDIATE_PAYMENT_REQUIRED"
    },
    "return_url": "https://example.com/returnUrl",
    "cancel_url": "https://example.com/cancelUrl"
  }
}'


- `plan_id`: 计划 ID
- `start_time`: 开始时间
- `quantity`: 产品数量
- `shipping_amount`: 运费
- `application_context`: 应用上下文

根据获取的结果选择 "links" 下第一个 "href" 去支付订阅。

### 6. 订阅详情

bash
curl -v -X GET https://api-m.sandbox.paypal.com/v1/billing/subscriptions/I-BW452GLLEP1G \
-H "Content-Type: application/json" \
-H "Authorization: Bearer Access-Token"


### 7. 配置 WebHook

此处选择的为个人选择。

👉 [野卡 | 一分钟注册，轻松订阅海外线上服务](https://bit.ly/bewildcard)