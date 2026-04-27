# 通知对象清单

本文档整理当前提供给清结算/外部系统查看的消息通知对象、所属 topic、通知方式、代码位置和字段结构。

结论先说：

- 这几个通知对象都定义在 `mdl/fund-catering-front/fund-catering-front-api`
- 也就是都属于 `front api` 模块
- 当前整理范围：
  - 扣款通知
  - 划付通知
  - 提现通知
  - 实收通知

## 1. 扣款通知

- 通知方式：API 通知
- topic：`mq_http_catering_deduction`
- 对象：`ConsumeNotifyDto`
- 代码位置：
  - 对象定义：[ConsumeNotifyDto.java](../../mdl/fund-catering-front/fund-catering-front-api/src/main/java/com/chinaums/erp/slhy/catering/front/message/ConsumeNotifyDto.java)
  - 消费者实现：[HttpDeductionMessageConsumeHandle.java](../../mdl/fund-catering-front/fund-catering-front-service/src/main/java/com/chinaums/erp/slhy/catering/front/handle/impl/message/HttpDeductionMessageConsumeHandle.java)

字段：

```java
private String code;
private String bizSeqNo;
private String msg;
private String subAccountType;
private String transAmt;
private String comment;
private String cardCode;
private String transNo;
private String transType;
private String transTime;
private String activityCode;
private String userId;
private String status;
private String userBusinessType;
private String userBusinessCode;
```

说明：

- 已去掉无效字段：
  - `bankSsn`
  - `bankAccNo`
  - `bankAccName`
  - `bankRemark`

## 2. 划付通知

- 通知方式：API 通知
- topic：`mq_http_catering_transfer`
- 对象：`TransferNotifyDto`
- 代码位置：
  - 对象定义：[TransferNotifyDto.java](../../mdl/fund-catering-front/fund-catering-front-api/src/main/java/com/chinaums/erp/slhy/catering/front/message/TransferNotifyDto.java)
  - 消费者实现：[HttpTransferMessageConsumeHandle.java](../../mdl/fund-catering-front/fund-catering-front-service/src/main/java/com/chinaums/erp/slhy/catering/front/handle/impl/message/HttpTransferMessageConsumeHandle.java)

字段：

```java
private String code;
private String bizSeqNo;
private String msg;
private String subAccountType;
private String transAmt;
private String comment;
private String bankSsn;
private String cardCode;
private String transNo;
private String transType;
private String transTime;
private String activityCode;
private String userId;
private String status;
private String userBusinessType;
private String userBusinessCode;
```

## 3. 提现通知

- 通知方式：HTTP 通知外部系统
- topic：`mq_http_catering_withDraw`
- 顶层对象：`WithDrawNotifyDto`
- 明细对象：`WithDrawNotifyItemDto`
- 代码位置：
  - 顶层对象：[WithDrawNotifyDto.java](../../mdl/fund-catering-front/fund-catering-front-api/src/main/java/com/chinaums/erp/slhy/catering/front/message/WithDrawNotifyDto.java)
  - 明细对象：[WithDrawNotifyItemDto.java](../../mdl/fund-catering-front/fund-catering-front-api/src/main/java/com/chinaums/erp/slhy/catering/front/message/WithDrawNotifyItemDto.java)
  - 组包实现：[WithDrawNotifyBuilder.java](../../mdl/fund-catering-task/src/main/java/com/chinaums/erp/slhy/catering/task/service/support/WithDrawNotifyBuilder.java)
  - 消费者实现：[HttpWithDrawMessageConsumeHandle.java](../../mdl/fund-catering-front/fund-catering-front-service/src/main/java/com/chinaums/erp/slhy/catering/front/handle/impl/message/HttpWithDrawMessageConsumeHandle.java)

### 3.1 顶层结构

```java
private String batchNo;
private Integer currentPage;
private Integer totalPages;
private List<WithDrawNotifyItemDto> dataList;
```

### 3.2 明细结构

```java
private String merchantId;
private String accountNo;
private String receiverAccount;
private String withdrawType;
private String beforeBalance;
private String afterBalance;
private String orderNo;
private String bankSerialNo;
private String type;
private String status;
private BigDecimal amount;
private String failReason;
private String txnTime;
private String finishTime;
```

### 3.3 最新字段要求

| 参数名 | 类型 | 是否必填 | 说明 |
|------|------|------|------|
| `merchantId` | `String` | 是 | 商户号，银商系统分配的商户号 |
| `accountNo` | `String` | 是 | 持牌人编号，当前业务口径使用 `cardCode` |
| `receiverAccount` | `String` | 是 | 持牌人收款账户号 |
| `withdrawType` | `String` | 是 | 自动提现类型，`01`-手动提现，`02`-自动提现 |
| `beforeBalance` | `String` | 是 | 交易前总可用余额；当前文档口径按账户变动明细表期初金额取值 |
| `afterBalance` | `String` | 是 | 交易后总可用余额；当前文档口径按账户变动明细表期末金额取值 |
| `orderNo` | `String` | 否 | FCT 发起提现时生成的提现订单号，按需求原文保留“`withdrawType=02 手动提现时必填`”描述，实际联调时需再确认该条件表述 |
| `bankSerialNo` | `String` | 否 | 银商流水号，`status=00` 成功时必填 |
| `type` | `String` | 否 | 提现类型，麦当劳提现专用字段。`00`-40%代扣提现，`01`-月度补扣款提现，`99`-其他 |
| `status` | `String` | 是 | 提现状态。`00`：成功，`01`：失败 |
| `amount` | `Number` | 是 | 提现金额，单位：元，保留 2 位小数 |
| `failReason` | `String` | 否 | 失败原因描述，当 `status=01` 失败时提供 |
| `txnTime` | `String` | 是 | 提现发起时间，格式 `YYYY-MM-DD HH:MM:SS` |
| `finishTime` | `String` | 否 | 提现完成时间，格式 `YYYY-MM-DD HH:MM:SS`，当 `status=00` 成功时提供 |

### 3.4 文档目标口径

- `accountNo`：
  - 按最新字段要求，应输出 `cardCode`
  - 不再按银行卡号/解密 `accountNoEnc` 口径理解
- `beforeBalance`：
  - 按账户变动明细表期初金额取值
  - 对应账户变动明细字段：
    - `orgAmt`
- `afterBalance`：
  - 按账户变动明细表期末金额取值
  - 当前文档目标口径对应账户变动明细字段：
    - `balance`

### 3.5 当前代码现状

- `batchNo`：雪花 ID
- `currentPage`：`1`
- `totalPages`：`1`
- `accountNo`：
  - 当前组包代码取的是提现子记录里的 `accountNoEnc`，并尝试解密后输出
  - 与最新字段要求中的 `cardCode` 口径不一致
- `receiverAccount`：
  - 当前组包代码取的是提现付款卡 `cardCode`
- `withdrawType`：
  - `FLWD / BAWD / FIWD` -> `02`
  - 其他提现 -> `01`
- `type`：直接按 `userBusinessCode` 输出
  - `WD`
  - `FLWD`
  - `BAWD`
  - `FIWD`
- `status`：
  - 成功 -> `00`
  - 失败 -> `01`
- `amount`：单位元，保留 2 位
- `beforeBalance`：
  - 当前代码取账户变动明细 `orgAmt`
  - 这与最新字段要求一致
- `afterBalance`：
  - 当前代码优先取 `realBalance`，否则回退 `balance`
  - 与“直接取账户变动明细表期末金额”的最新口径不完全一致
- `finishTime`：成功时返回

### 3.6 对外实际发送 body

提现通知消费者对外只发送下面 4 个顶层字段，不发送旧单笔字段：

```json
{
  "batchNo": "...",
  "currentPage": 1,
  "totalPages": 1,
  "dataList": [
    {
      "merchantId": "...",
      "accountNo": "...",
      "receiverAccount": "...",
      "withdrawType": "...",
      "beforeBalance": "...",
      "afterBalance": "...",
      "orderNo": "...",
      "bankSerialNo": "...",
      "type": "...",
      "status": "...",
      "amount": 0.00,
      "failReason": "...",
      "txnTime": "...",
      "finishTime": "..."
    }
  ]
}
```

## 4. 实收通知

- 通知方式：API 通知
- topic：`mq_http_catering_actual_receipt`
- 对象：`ActualReceiptNotifyDto`
- 代码位置：
  - 对象定义：[ActualReceiptNotifyDto.java](../../mdl/fund-catering-front/fund-catering-front-api/src/main/java/com/chinaums/erp/slhy/catering/front/message/ActualReceiptNotifyDto.java)
  - 消费者实现：[HttpActualReceiptMessageConsumeHandle.java](../../mdl/fund-catering-front/fund-catering-front-service/src/main/java/com/chinaums/erp/slhy/catering/front/handle/impl/message/HttpActualReceiptMessageConsumeHandle.java)
  - 发送入口：[PlatformRechargeJobService.java](../../mdl/fund-catering-task/src/main/java/com/chinaums/erp/slhy/catering/task/job/zx/PlatformRechargeJobService.java)

字段：

```java
private String code;
private String msg;
private String status;
private String transNo;
private String remittanceId;
private String merchantId;
private String bankSerialNo;
private String payAcctNo;
private String payAcctNm;
private String acctNo;
private String transAmt;
private String transDate;
private String timeStampe;
private String transJrno;
private String remittanceNote;
private String processingStatus;
private String processingMethod;
private String bankProcessingSerialNo;
private String bankProcessingCompletionTime;
private String bankResult;
private String remark;
```

说明：

- 实收通知当前已经改为基于 `PlatformRechargeJobService`
- 不再基于“不明来款 / UnknownTrans”链路推送

## 5. topic 清单

- 扣款：`mq_http_catering_deduction`
- 划付：`mq_http_catering_transfer`
- 提现：`mq_http_catering_withDraw`
- 实收：`mq_http_catering_actual_receipt`

## 6. bas_param_t 参数清单

当前相关通知参数：

- `deductionNotifyUrl`
- `transferNotifyUrl`
- `withdrawNotifyUrl`
- `withdrawNotifySysId`
- `withdrawNotifyAuthorization`
- `withdrawNotifyVersion`
- `actualReceiptNotifyUrl`

说明：

- 提现通知鉴权头从 `bas_param_t` 获取
- 实收通知地址从 `bas_param_t` 获取
- 其他通知地址也统一按参数表口径处理
