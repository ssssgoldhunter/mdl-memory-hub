# A/B/C/D 与 front 对接 Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 完成 `A/B/C/D` 与 `front` 的清结算对接改造，落地提现前规则校验、扣款/划付通知、原冻结能力边界与 `front` 1 小时重发机制。

**Architecture:** 以 `front` 作为统一清结算对接出口。`A` 在提现主流程中新增 LiteFlow 校验节点并通过 `front` 查询清结算；`B`、`D` 在业务侧完成各自后置处理后投递到 `front`，由 `front` 执行 HTTP/Feign 通知与 1 小时重发；`C` 保持独立 Web API / service 能力，不进入异步通知模型。

**Tech Stack:** Java 17, Spring Boot, LiteFlow, OpenFeign, RocketMQ, Redis, MyBatis Plus

---

## 0. 计划输入

**已确认文档：**

- `mdl-memory-hub/requirements/2026-03-26-abcd-front-design-confirmed.md`
- `mdl-memory-hub/requirements/overview.md`
- `mdl-memory-hub/requirements/req-b-deduction-api/summary.md`
- `mdl-memory-hub/requirements/req-c-frozen-refactor/summary.md`
- `mdl-memory-hub/requirements/req-d-transfer-mode-refactor/summary.md`

**本计划输出目标：**

- `A`：提现前规则校验设计落地
- `B`：扣款接口与 `D + 04` 异步上账支持设计落地
- `C`：冻结/解冻能力边界确认并体现在接口设计中
- `D`：`02` 模式 task 执行与通知时机落地
- `front`：清结算查询与 `B/D` 通知重发机制落地

---

### Task 1: A 提现前规则校验落点设计

**Files:**
- Modify: `bwcj/fund-catering-web` 下提现 Web 请求对象、提现入口 controller、请求转换逻辑相关文件
- Modify: `bwcj/fund-catering-consume` 或提现链路所在服务中的 LiteFlow 配置与节点实现文件
- Create: 提现规则校验节点实现类
- Create: `front` 查询清结算能力所需 DTO / facade / service 文件
- Test: 提现链路相关单测或集成验证用例文件

- [ ] **Step 1: 定位提现 Web 请求对象与 `needRuleCheck` 落点**

输出：
- 找到提现 Web 对外请求对象
- 明确 `needRuleCheck` 字段新增位置
- 明确参数校验规则

- [ ] **Step 2: 定位提现 LiteFlow 主链路和现有校验节点顺序**

输出：
- 找到提现主链路定义
- 明确新增节点插入位置为“基础参数校验后、正式提现前”

- [ ] **Step 3: 设计并新增提现规则校验节点**

实现要求：
- 节点内部先判断 `needRuleCheck`
- `false` 时直接放行
- `true` 时调用 `front` 查询清结算
- 查到卡号存在扣款失败数据则抛出业务失败

- [ ] **Step 4: 在 `front` 增加清结算查询入口**

实现要求：
- 对业务侧暴露稳定调用入口
- 当前调用方式按 `HTTP/Feign` 暂定
- 保留后续适配清结算正式接口返回字段的扩展点

- [ ] **Step 5: 为 A 链路补验证用例**

至少覆盖：
- `needRuleCheck=false` 不触发查询
- `needRuleCheck=true` 且查无失败数据时放行
- `needRuleCheck=true` 且查到失败数据时拦截

---

### Task 2: B 新扣款请求对象与主链路改造设计

**Files:**
- Create: `bwcj/fund-catering-web` 中 `B` 专用 Web 请求对象
- Modify: `bwcj/fund-catering-web` 中扣款入口 controller / request convert 文件
- Modify: `bwcj/fund-catering-consume` 中扣款主流程入口、参数校验、trans/after 相关文件
- Test: `B` 请求转换与主流程校验测试文件

- [ ] **Step 1: 新建 B 专用请求对象**

要求：
- 字段命名尽量复用面密消费对象
- 只保留付款卡、收款卡两个卡号字段
- 删除分账、多账户类型相关字段
- 只支持 `04`

- [ ] **Step 2: 明确 `useFrozen` / `oldFrozenTransNo` 参数校验**

要求：
- `useFrozen=true` 时 `oldFrozenTransNo` 必传
- `useFrozen=false` 时不得误入原冻结额度池逻辑

- [ ] **Step 3: 接入 B 的 controller / service 转换逻辑**

要求：
- Web 入参转换到内部请求对象
- 保持现有消费字段映射兼容性

- [ ] **Step 4: 明确 B 主链路分支**

要求：
- `useFrozen=false` 走普通 `04` 扣款路径
- `useFrozen=true` 进入独立冻结扣款分支

- [ ] **Step 5: 补 B 的请求校验与主分支验证**

至少覆盖：
- `useFrozen=false`
- `useFrozen=true + oldFrozenTransNo`
- 非 `04` 或非法字段输入失败

---

### Task 3: B after 与异步上账 `D + 04` 支持

**Files:**
- Modify: `bwcj/fund-catering-consume` 中 `B after`、账户处理、冻结明细处理相关文件
- Modify: `bwcj/fund-catering-consume` 中异步上账枚举、分支、服务实现相关文件
- Test: `B after` 和异步上账验证文件

- [ ] **Step 1: 定位普通 `04` after 与冻结扣款 after 的复用边界**

输出：
- 明确 `useFrozen=false` 继续复用现有普通 `04 after`
- 明确 `useFrozen=true` 独立 `B after`

- [ ] **Step 2: 实现 `useFrozen=true` 的 `B after`**

处理顺序要求：
- 扣 `balance`
- 减 `frozenAmt`
- 写 `trans_acct_frozen_change_detail_t`
- 衔接异步上账

- [ ] **Step 3: 定位异步上账现有交易类型判断逻辑**

输出：
- 找到交易类型枚举
- 找到 `04` 上账入口
- 找到需要补 `D` 的代码点

- [ ] **Step 4: 补齐 `D + 04` 的异步上账支持**

要求：
- 只补 `D + 04`
- 不扩其他交易类型
- 不引入多余抽象

- [ ] **Step 5: 补 B after / 异步上账验证**

至少覆盖：
- `useFrozen=true` 成功路径
- `D + 04` 异步上账被识别

---

### Task 4: C 独立冻结/解冻能力边界落地

**Files:**
- Modify: `bwcj/fund-catering-web` 中冻结/解冻 Web API 入口文件
- Modify: `bwcj/fund-catering-consume` 或相关服务中 `C` 的 service 接口、after、通用冻结能力文件
- Test: `C` 的接口与能力边界验证文件

- [ ] **Step 1: 梳理 C 的 Web API 与 service 接口边界**

输出：
- 外部用户通过 Web API 调用冻结 / 解冻
- 清结算通过独立 service 接口调用冻结 / 解冻

- [ ] **Step 2: 确认 C 的公共冻结额度控制层责任**

要求：
- 仅负责 `trans_frozen_t`
- 不负责账户金额和冻结明细

- [ ] **Step 3: 确认 C after 的账户处理边界**

要求：
- `C after` 负责冻结金额增减
- 不进入 `front` 异步通知体系

- [ ] **Step 4: 补 C 边界验证**

至少覆盖：
- Web API 路径
- service 接口路径
- 不触发 front 通知

---

### Task 5: D `02` 模式接口受理与 task 执行改造

**Files:**
- Modify: `bwcj/fund-catering-web` 中划付入口 controller / request convert 文件
- Modify: `bwcj/fund-catering-consume` 或相关业务模块中划付明细新增接口文件
- Modify: `bwcj/fund-catering-task` 中 `02` 模式 task、状态推进、补偿相关文件
- Test: `D` 受理与 task 执行验证文件

- [ ] **Step 1: 定位 `01` 现有划付明细新增接口**

输出：
- 明确 `02` 复用入口
- 明确需要新增/修改的字段

- [ ] **Step 2: 设计 `02` 模式接口受理阶段落库**

要求：
- 接口只落待处理记录
- 不做账户变动
- 不做冻结主记录写入

- [ ] **Step 3: 实现 `02` 模式 task 扫描与执行入口**

要求：
- 扫描待处理数据
- 加锁防并发
- 执行实际内部上下账
- 推进状态

- [ ] **Step 4: 实现 `isFrozen=true` 的冻结写入时机**

要求：
- 账户实际完成划付后再写 `F`
- 不在接口受理时预写

- [ ] **Step 5: 补 D `02` 模式验证**

至少覆盖：
- 受理只落记录
- task 成功执行
- `isFrozen=true` 后写 `F`

---

### Task 6: B/D -> front 通知触发链路改造

**Files:**
- Modify: `bwcj/fund-catering-consume` / `bwcj/fund-catering-task` 中触发 `front` 通知的业务文件
- Modify: `bwcj/fund-catering-front` 中消息类型枚举、DTO、发送入口相关文件
- Test: `B/D` 通知触发链路验证文件

- [ ] **Step 1: 明确 B 通知触发代码点**

要求：
- 异步上账完成后触发
- 补偿完成后也触发

- [ ] **Step 2: 明确 D 通知触发代码点**

要求：
- task 实际划付成功后触发
- 补偿完成后也触发

- [ ] **Step 3: 为 B/D 新增独立通知类型**

要求：
- 消息类型分开
- DTO 分开
- 但处理实现优先复用现有消费 / 转账能力

- [ ] **Step 4: 补 B/D 通知触发验证**

至少覆盖：
- B 正常触发
- D 正常触发
- 补偿完成后二次触发

---

### Task 7: front 清结算查询与 1 小时重发改造

**Files:**
- Modify: `bwcj/fund-catering-front` 中 facade、controller、service、consumer、消息日志、重发控制相关文件
- Modify: `bwcj/fund-catering-front` 中 `MessageTypeTopicEnum`、DTO、HTTP 调用处理类
- Test: `front` 查询与重发机制验证文件

- [ ] **Step 1: 设计并实现 A 的清结算查询入口**

要求：
- 提供稳定查询方法给业务系统调用
- 当前协议按 `HTTP/Feign` 暂定
- 对返回成功语义保留扩展点

- [ ] **Step 2: 梳理现有 front 重发机制**

输出：
- 当前 Redis 计数逻辑
- 当前最大重试次数逻辑
- 当前消息日志能力

- [ ] **Step 3: 改造 B/D 的重发控制为 1 小时窗口**

要求：
- 记录首次发送时间
- 1 小时内持续重发
- 超时登记失败
- 不再被原有限次重试直接截断

- [ ] **Step 4: 实现 B/D 清结算 HTTP 调用处理**

要求：
- 保持 HTTP 模式
- 成功判定逻辑可后续扩展
- 当前先按“明确成功才算成功”原则预留接口

- [ ] **Step 5: 补 front 查询与重发验证**

至少覆盖：
- 查询接口调用成功
- B/D 通知进入重发
- 1 小时超时后登记失败

---

### Task 8: 文档、联调与回归验证

**Files:**
- Modify: `mdl-memory-hub/requirements/*.md`
- Modify: `mdl-memory-hub/docs/plans/*.md`
- Test: 各模块联调记录或验证清单

- [ ] **Step 1: 同步更新设计文档**

要求：
- 若实现细节调整，回写需求摘要和确认稿

- [ ] **Step 2: 梳理联调清单**

至少包括：
- A 查询链路
- A 提现结果通知
- B 异步上账后通知
- C Web/API 与 service 边界
- D task 成功后通知
- front 1 小时重发

- [ ] **Step 3: 运行模块级验证**

要求：
- 至少覆盖受影响模块编译 / 测试
- 记录未执行项

- [ ] **Step 4: 整理最终验证结论**

输出：
- 通过项
- 待确认项
- 依赖清结算接口最终定义的残留点
