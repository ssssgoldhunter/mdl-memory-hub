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
  - 对象定义：[ConsumeNotifyDto.java](/Users/limeng/workspaces/IdeaProjects_mdl_dep/mdl/fund-catering-front/fund-catering-front-api/src/main/java/com/chinaums/erp/slhy/catering/front/message/ConsumeNotifyDto.java)
  - 消费者实现：[HttpDeductionMessageConsumeHandle.java](/Users/limeng/workspaces/IdeaProjects_mdl_dep/mdl/fund-catering-front/fund-catering-front-service/src/main/java/com/chinaums/erp/slhy/catering/front/handle/impl/message/HttpDeductionMessageConsumeHandle.java)

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
  - 对象定义：[TransferNotifyDto.java](/Users/limeng/workspaces/IdeaProjects_mdl_dep/mdl/fund-catering-front/fund-catering-front-api/src/main/java/com/chinaums/erp/slhy/catering/front/message/TransferNotifyDto.java)
  - 消费者实现：[HttpTransferMessageConsumeHandle.java](/Users/limeng/workspaces/IdeaProjects_mdl_dep/mdl/fund-catering-front/fund-catering-front-service/src/main/java/com/chinaums/erp/slhy/catering/front/handle/impl/message/HttpTransferMessageConsumeHandle.java)

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
  - 顶层对象：[WithDrawNotifyDto.java](/Users/limeng/workspaces/IdeaProjects_mdl_dep/mdl/fund-catering-front/fund-catering-front-api/src/main/java/com/chinaums/erp/slhy/catering/front/message/WithDrawNotifyDto.java)
  - 明细对象：[WithDrawNotifyItemDto.java](/Users/limeng/workspaces/IdeaProjects_mdl_dep/mdl/fund-catering-front/fund-catering-front-api/src/main/java/com/chinaums/erp/slhy/catering/front/message/WithDrawNotifyItemDto.java)
  - 组包实现：[WithDrawNotifyBuilder.java](/Users/limeng/workspaces/IdeaProjects_mdl_dep/mdl/fund-catering-task/src/main/java/com/chinaums/erp/slhy/catering/task/service/support/WithDrawNotifyBuilder.java)
  - 消费者实现：[HttpWithDrawMessageConsumeHandle.java](/Users/limeng/workspaces/IdeaProjects_mdl_dep/mdl/fund-catering-front/fund-catering-front-service/src/main/java/com/chinaums/erp/slhy/catering/front/handle/impl/message/HttpWithDrawMessageConsumeHandle.java)

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

### 3.3 当前实现口径

- `batchNo`：雪花 ID
- `currentPage`：`1`
- `totalPages`：`1`
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
- `beforeBalance / afterBalance`：单位元，保留 2 位
- `finishTime`：成功时返回

### 3.4 对外实际发送 body

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
  - 对象定义：[ActualReceiptNotifyDto.java](/Users/limeng/workspaces/IdeaProjects_mdl_dep/mdl/fund-catering-front/fund-catering-front-api/src/main/java/com/chinaums/erp/slhy/catering/front/message/ActualReceiptNotifyDto.java)
  - 消费者实现：[HttpActualReceiptMessageConsumeHandle.java](/Users/limeng/workspaces/IdeaProjects_mdl_dep/mdl/fund-catering-front/fund-catering-front-service/src/main/java/com/chinaums/erp/slhy/catering/front/handle/impl/message/HttpActualReceiptMessageConsumeHandle.java)
  - 发送入口：[PlatformRechargeJobService.java](/Users/limeng/workspaces/IdeaProjects_mdl_dep/mdl/fund-catering-task/src/main/java/com/chinaums/erp/slhy/catering/task/job/zx/PlatformRechargeJobService.java)

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
