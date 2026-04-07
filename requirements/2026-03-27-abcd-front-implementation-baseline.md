# A/B/C/D 与 front 当前实现基线

更新时间：2026-03-27 14:10:00 CST

> 本文档后续不再作为主入口维护。  
> 当前请优先查看 `docs/superpowers/mdl-supply-chain-abcd/00-current-baseline.md`。
>
> 额外说明：本文档中关于 `B=transType P`、`D02=两接口执行`、`D02=data-batch 启动` 等内容，均已在 2026-03-31 后续纠偏中失效。

## 1. 文档目的

本文档用于覆盖 2026-03-26 确认稿之后，因多轮续聊、纠偏、编码落地而形成的最新实现边界。

本文档的角色不是替代历史设计讨论，而是明确：

- 当前代码已经按什么边界实现
- 哪些旧概念已经废弃
- 哪些能力仍待外部系统或测试补齐

优先级说明：

- 如与 `2026-03-26-abcd-front-design-confirmed.md` 冲突
- 以本文档为当前实现基线

---

## 2. 总体结论

### 2.1 当前主顺序已经走完

当前代码实现顺序已经按下列主线落地：

1. `A` 提现规则校验链路
2. `C` 原冻结额度池 / 冻结解冻能力
3. `B` 扣款主交易与 `useFrozen=true`
4. `D` 划付模式改造与 `02 task`
5. `front` 的 `B/D` 通知发送端收口

### 2.2 当前真正未完成项

- `A` 的清结算查询接口仍未提供，节点里是 `TODO`
- 自动化测试还没补
- 真实联调与数据验证仍待执行

### 2.3 已明确废弃的概念

- `A` 通过 `front` 先查本地失败扣款再拦截提现
- `B` 自建 `DeductionTransSlot`
- `D02` 对外暴露 `registerMode02Detail / batchRegisterMode02Detail`
- `C` 再新增“oldFrozenTransNo”类字段去承载原冻结流水号

---

## 3. 需求 A 当前实现边界

### 3.1 对外入参

- `scWithdraw` 已增加：
  - `needRuleCheck`

### 3.2 规则校验位置

- 提现 `LiteFlow` 已新增独立节点：
  - `withDrawRuleCheck`
- 位置：
  - 常规 check 之后
  - 正式提现主交易之前

### 3.3 当前真实行为

- `needRuleCheck=false`
  - 节点直接跳过
- `needRuleCheck=true`
  - 进入节点
  - 记录日志
  - 暂时放行

### 3.4 当前未完成项

- 节点中的清结算查询仍是 `TODO`
- 等清结算系统提供查询 API 后，补：
  - 按卡号查询失败扣款
  - 查到则拦截提现

### 3.5 front 边界

- `front` 当前不承担 `A` 的提现前规则查询
- `front` 在 `A` 中仍只承担：
  - 提现结果通知

---

## 4. 需求 B 当前实现边界

### 4.1 入口与核心流程边界

- `B` 的对外入口挂在现有消费体系中
- Web 入口：
  - `/api/ums/catering/trans/scDeduction`
- 但 `B` 核心交易流程与普通消费已经分开

### 4.2 当前代码口径

- `B` 复用原 `TransSlot`
- `B` 有独立 `deduction` 组件目录
- `B` 的核心交易、冻结池联动、after 不走普通消费主链
- 只复用：
  - 通用校验
  - 通用上下文
  - 现有通知基础设施

### 4.3 交易语义

- `B` 使用：
  - `transType = P`
- 当前只支持：
  - `04`

### 4.4 两种模式

#### `useFrozen=false`

- 走 `B` 自己的普通扣款主链
- 不进入原冻结额度池

#### `useFrozen=true`

- `oldFrozenTransNo` 必传
- 使用 `C` 的原冻结额度池能力
- 走独立 `B after`

### 4.5 通知边界

- `B` 成功后的异步通知已接到独立 topic
- `front` 使用独立 deduction topic 处理
- 处理实现继续复用现有 HTTP consumer 框架

---

## 5. 需求 C 当前实现边界

### 5.1 能力定位

- `C` 当前继续作为：
  - 原冻结额度池
  - 独立冻结能力
  - 独立解冻能力

### 5.2 原冻结流水号口径

- 解冻链路只使用：
  - `orgTransNo`
- 不新增额外“旧冻结流水号”字段

### 5.3 当前实现口径

- 冻结主记录继续使用业务主流水
- 解冻按原冻结流水号定位原记录
- 当前代码已支持：
  - 按剩余冻结额度做部分解冻
- `B useFrozen=true` 当前代码已支持：
  - 按剩余冻结额度做部分冻结扣款
- 当前冻结额度管理核心语义是：
  - 通过原冻结记录的 `frozenAmt / freezableAmt / allowFlag` 管理剩余额度
  - `C` 的解冻与 `B` 的冻结扣款共用这套剩余额度语义
- 原冻结全部释放后：
  - 原记录 `allowFlag` 关闭
- 原冻结额度被完全使用后：
  - 原记录 `allowFlag` 关闭

### 5.4 与 B/D 的关系

- `B useFrozen=true`
  - 复用 `C` 的原冻结额度池能力
  - 当前已按“部分冻结扣款”口径实现
- `D isFrozen=true`
  - 生成的新原冻结记录可被 `B/C` 后续继续使用

### 5.5 通知边界

- `C` 不走异步通知模型

---

## 6. 需求 D 当前实现边界

### 6.1 模式开关

- 当前优先从 `param_t` 读取：
  - `TRANSFER_MODE_SWITCH`
- 若参数表无值：
  - 再回退请求里的 `transferMode`
- 若仍无值：
  - 默认 `01`

### 6.2 `01` 模式

- 保持原有划付逻辑

### 6.3 `02` 模式接口阶段

- 接口阶段先走：
  - `transTransferTiPre`
  - `batchAddTransferTiData`
- `02` 只复用 `01` 中间那段批量上传明细能力
- 不再对外暴露：
  - `createBatchTask`
  - `triggerBatchTask`
  - `registerMode02Detail`
  - `batchRegisterMode02Detail`

### 6.4 `02` 模式服务内部处理

- 为了让接口侧只使用中间能力
- service 内部允许在 `SC_D02_` 批次号下自动建内部批次
- 这属于服务内部兜底，不改变接口语义

### 6.5 `02` 模式 task 阶段

- 真正划付执行仍由 task 完成
- task 负责：
  - 实际上下账
  - 状态推进
  - 冻结池联动
  - 成功后的通知发送

### 6.6 `useFrozen / isFrozen`

- `useFrozen=true`
  - task 执行时消费原冻结额度池
- `isFrozen=true`
  - task 真正划付成功后，写新的原冻结主记录 `F`
- `02` 任务侧透传：
  - `useFrozen`
  - `oldFrozenTransNo`
  - `isFrozen`
- 当前通过任务明细里的业务标签元数据承载

### 6.7 通知边界

- `D` 不在接口受理成功后立即通知清结算
- 当前在 task 成功点发送到 `front`
- `front` 使用独立 transfer topic

---

## 7. front 当前实现边界

### 7.1 A 查询边界

- 当前不负责 `A` 的提现前查询
- `A` 查询真实接入点在 `withDrawRuleCheck`，但仍待清结算 API

### 7.2 B / D 通知

- `B`
  - `TOPIC_NOTIFY_CATERING_DEDUCTION`
- `D`
  - `TOPIC_NOTIFY_CATERING_TRANSFER`

### 7.3 重发机制

- 继续沿用现有 HTTP consumer 框架
- 在 `1` 小时窗口内持续重发
- 超过 `1` 小时不再继续调度

---

## 8. 当前待补项

### 8.1 必须等外部条件满足后再补

- `A` 的清结算查询真实接口

### 8.2 仍待验证

- `B useFrozen=true` 数据级联动
- `D 02 + useFrozen=true` task 行为
- `D 02 + isFrozen=true` 新原冻结写入
- `front` 通知失败后的 1 小时重发

### 8.3 仍待补充

- 自动化测试
