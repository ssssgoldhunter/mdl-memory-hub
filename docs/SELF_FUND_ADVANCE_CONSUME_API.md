# 自有资金池垫资 Consume 接口调用文档

> Status: current, verified-against-source
> Last updated: 2026-05-12

本文档给清结算系统调用 `fund-catering-consume` 使用，覆盖自有资金池垫资里的两类接口：

- 清结算垫资：内部转账接口
- 垫资扣款：平台收款接口

## 1. 公共说明

| 项 | 说明 |
|----|------|
| Feign Client | `com.chinaums.erp.slhy.catering.consume.api.TransConsumeApi` |
| 服务名 | `fund-catering-consume` |
| 基础路径 | `/consume/trans` |
| 请求方式 | `POST` |
| 请求体 | JSON |
| 金额单位 | 入参 `amount` 为分，整数字符串，例如 `12345` 表示 123.45 元 |
| 返回金额 | 返回 `model.amount`、`model.balance` 通常为元格式字符串，由服务端 `NumberUtils.getTruncStr(...)` 转换 |
| 幂等键 | `transNo`，调用方必须保证业务侧全局唯一 |
| 账户类型 | 本业务只使用 `accountType=04` |

## 2. 内部转账接口

### 2.1 Feign 方法

```java
@PostMapping(value = "/consume/trans/transTransferInner")
DefaultResult<TransferTransRes> transTransferInner(@RequestBody ConsumeTransferRequest request) throws Exception;
```

### 2.2 使用场景

清结算垫资使用。银行侧资金动作由银行完成，本接口只做系统内部账户记账：

- 付款方：垫资资金来源卡 `payCardCode`
- 收款方：平台或业务收款卡 `receiveCardCode`
- 交易类型：服务内部设置为 `AT`

### 2.3 请求字段

| 字段 | 必填 | 单位/取值 | 说明 |
|------|------|-----------|------|
| `merchantId` | 是 | 字符串 | 商户号 |
| `payCardCode` | 是 | 字符串 | 付款方卡号 |
| `receiveCardCode` | 是 | 字符串 | 收款方卡号，不能和付款方相同 |
| `txnTime` | 是 | 字符串 | 交易时间，沿用现有系统格式 |
| `amount` | 是 | 分 | 整数字符串，必须大于 0 |
| `transNo` | 是 | 字符串 | 清结算侧业务流水号，必须唯一 |
| `accountType` | 建议传 | `04` | 不传时 DTO 默认 `04`，本业务固定使用 `04` |
| `payAccChangeType` | 建议传 | `AC` | 付款方账户变动类型，便于明细口径识别 |
| `recAccChangeType` | 建议传 | `AR` | 收款方账户变动类型，便于明细口径识别 |
| `transBusType` | 可选 | 字符串 | 业务转账类型，可按清结算业务口径传 |
| `remark` | 可选 | 字符串 | 备注 |
| `terminalCode` | 可选 | 字符串 | 终端编码 |
| `userBusinessType` | 可选 | 字符串 | 用户业务类型 |
| `userBusinessCode` | 可选 | 字符串 | 用户业务编码 |
| `userBusinessCommon` | 可选 | 字符串 | 用户业务通用字段 |
| `userBusinessTags` | 可选 | 字符串 | 用户业务标签 |

### 2.4 请求示例

```json
{
  "merchantId": "MERCHANT001",
  "payCardCode": "SELF_FUND_CARD_CODE",
  "receiveCardCode": "PLATFORM_RECEIVE_CARD_CODE",
  "txnTime": "2026-05-12 10:30:00",
  "amount": "12345",
  "transNo": "SETTLE_ADVANCE_202605120001",
  "accountType": "04",
  "payAccChangeType": "AC",
  "recAccChangeType": "AR",
  "transBusType": "C",
  "remark": "清结算自有资金池垫资"
}
```

### 2.5 返回模型关键字段

| 字段 | 说明 |
|------|------|
| `model.bizSeqNo` | 系统交易流水号 |
| `model.transNo` | 调用方交易单号 |
| `model.amount` | 交易金额，元格式字符串 |
| `model.payCardCode` | 付款方卡号 |
| `model.receiveCardCode` | 收款方卡号 |
| `model.status` | 交易状态 |
| `model.subAccDetail` | 子账户明细 |

## 3. 平台收款接口

### 3.1 Feign 方法

```java
@PostMapping(value = "/consume/trans/transPlatformReceive")
DefaultResult<TransConsumeRes> transPlatformReceive(@RequestBody ConsumeDeductionTransRequest request) throws Exception;
```

`transPlatformDeduction(...)` 当前实现为直接复用 `transPlatformReceive(...)`。清结算系统建议直接调用 `transPlatformReceive(...)`，语义更清楚。

### 3.2 使用场景

垫资扣款使用。通过新的平台收款账号，完成平台自有资金账户收款：

- 付款方：业务方卡 `payCardCode`
- 收款方：平台收款卡 `receiveCardCode`
- 平台账户：由 `platformCardCode`、`platformBankEAccountId`、`platformDealType`、`platformFundTp` 指定
- 交易类型：服务内部设置为 `MR`

### 3.3 请求字段

| 字段 | 必填 | 单位/取值 | 说明 |
|------|------|-----------|------|
| `merchantId` | 是 | 字符串 | 商户号 |
| `payCardCode` | 是 | 字符串 | 付款方业务卡号 |
| `receiveCardCode` | 是 | 字符串 | 收款方平台卡号，不能和付款方相同 |
| `txnTime` | 是 | 字符串 | 交易时间，沿用现有系统格式 |
| `amount` | 是 | 分 | 整数字符串，必须大于 0 |
| `transNo` | 是 | 字符串 | 清结算侧业务流水号，必须唯一 |
| `accountType` | 可选 | `04` | 服务内部会设置为 `04`，调用方可显式传入 |
| `platformCardCode` | 是 | 字符串 | 平台自有资金卡号 |
| `platformBankEAccountId` | 是 | 字符串 | 平台银行电子账号 ID |
| `platformRegisterType` | 可选 | `12` | 平台注册类型，未传时按业务配置口径处理 |
| `platformDealType` | 是 | 字符串 | 平台交易处理类型 |
| `platformFundTp` | 是 | 字符串 | 平台资金类型 |
| `remark` | 可选 | 字符串 | 备注 |
| `orderId` | 可选 | 字符串 | 业务订单号 |
| `terminalId` | 可选 | 字符串 | 终端号 |
| `goodsTag` | 可选 | 字符串 | 商品标识 |
| `comment` | 可选 | 字符串 | 公用字段 |
| `userBusinessType` | 可选 | 字符串 | 用户业务类型 |
| `userBusinessCode` | 可选 | 字符串 | 用户业务编码 |
| `userBusinessCommon` | 可选 | 字符串 | 用户业务通用字段 |
| `userBusinessTags` | 可选 | 字符串 | 用户业务标签 |

### 3.4 请求示例

```json
{
  "merchantId": "MERCHANT001",
  "payCardCode": "BUSINESS_PAY_CARD_CODE",
  "receiveCardCode": "PLATFORM_RECEIVE_CARD_CODE",
  "txnTime": "2026-05-12 10:35:00",
  "amount": "12345",
  "transNo": "SETTLE_ADVANCE_DEDUCT_202605120001",
  "accountType": "04",
  "platformCardCode": "SELF_FUND_CARD_CODE",
  "platformBankEAccountId": "PLATFORM_BANK_E_ACCOUNT_ID",
  "platformRegisterType": "12",
  "platformDealType": "DEAL_TYPE",
  "platformFundTp": "FUND_TP",
  "remark": "清结算自有资金池垫资扣款"
}
```

### 3.5 返回模型关键字段

| 字段 | 说明 |
|------|------|
| `model.bizSeqNo` | 系统交易流水号 |
| `model.transNo` | 调用方交易单号 |
| `model.orderId` | 业务订单号 |
| `model.amount` | 交易金额，元格式字符串 |
| `model.balance` | 付款方 `04` 子账户余额，元格式字符串 |
| `model.transStatus` | 订单状态 |
| `model.payCardCode` | 付款方卡号 |

## 4. 前置配置

本业务至少需要以下配置，否则接口可能在打包、卡 BIN 权限或平台账户校验阶段失败：

1. `bas_param_t`
   - 参数编码：`SELF_FUND_ACCOUNT_CONFIG`
   - 配置内容包含：
     - `cardcode`
     - `ebankid`
     - `registerType`，默认 `12`
     - `dealType`
     - `fundTp`

2. `bas_card_bin_business_t`
   - 自有资金池卡 BIN 需要开通 `MC`，用于平台付款。
   - 业务付款卡 BIN 需要开通 `MR`，用于平台收款/扣款。
   - 缺失时会报卡 BIN 业务信息不存在。

## 5. 调用注意事项

- `amount` 入参是分，不是元。
- `payCardCode` 和 `receiveCardCode` 不能相同。
- `transNo` 重复时，内部转账会按重复流水失败；平台收款存在幂等检查，但仍要求调用方按唯一流水调用。
- 平台收款会校验 `platformCardCode`、`platformBankEAccountId`、`platformDealType`、`platformFundTp` 非空。
- 平台收款只锁付款方卡号；内部转账同时锁付款方和收款方卡号。
- 清结算垫资不要调用普通转账 `/transTransfer`，应调用内部转账 `/transTransferInner`。
- 垫资扣款建议调用 `/transPlatformReceive`，不要用普通扣款 `/transDeduction`。

## 6. 源码依据

- `../mdl/fund-catering-consume/fund-catering-consume-api/src/main/java/com/chinaums/erp/slhy/catering/consume/api/TransConsumeApi.java`
- `../mdl/fund-catering-consume/fund-catering-consume-api/src/main/java/com/chinaums/erp/slhy/catering/consume/request/ConsumeTransferRequest.java`
- `../mdl/fund-catering-consume/fund-catering-consume-api/src/main/java/com/chinaums/erp/slhy/catering/consume/request/ConsumeDeductionTransRequest.java`
- `../mdl/fund-catering-consume/fund-catering-consume-api/src/main/java/com/chinaums/erp/slhy/catering/consume/request/ConsumeTransFreeRequest.java`
- `../mdl/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/service/impl/TransTransferServiceImpl.java`
- `../mdl/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/service/impl/TransConsumeServiceImpl.java`
- `../mdl/fund-catering-consume/fund-catering-consume-service/src/main/java/com/chinaums/erp/slhy/catering/consume/flow/component/trans/platform/AbstractPlatformTransPack.java`
