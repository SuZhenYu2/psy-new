# PsyAssess 学校心理健康管理平台 — AI优先设计文档

> 日期: 2026-05-18 | 状态: 草稿 | 模式: AI优先

---

## 一、问题陈述

学校心理健康工作面临以下痛点：
1. **工具分散**：心理老师使用多个零散工具（纸质问卷、Excel、聊天工具）拼凑完成工作
2. **效率低下**：人工统计分析耗时，无法及时发现风险学生
3. **报告质量参差不齐**：依赖心理老师个人经验，缺乏标准化分析
4. **预警滞后**：无法实时监测学生心理状态变化

## 二、需求证据

- **实际使用数据**：已有试点学校正在使用类似工具，活跃度高
- **用户依赖**：心理老师围绕现有工具构建工作流程
- **痛点明确**：访谈显示心理老师最需要自动化报告生成和智能预警

## 三、核心用户画像

### 心理老师（核心用户）
- **职位**：学校专职心理老师
- **升职关键**：有效预防心理危机、提升学生心理健康水平
- **风险**：未能及时发现有自伤倾向的学生
- **痛点**：深夜担心学生安全、报告撰写占用大量时间

### 其他角色
| 角色 | 核心需求 |
|------|----------|
| 班主任 | 查看班级学生状态、接收预警通知 |
| 校级管理员 | 学校层面统计分析、报表导出 |
| 教育局管理员 | 区域数据汇总、趋势分析 |
| 学生 | 便捷完成测评、查看个人报告 |
| 家长 | 接收学生报告、了解孩子状态 |

## 四、最小可行版本（AI优先）

### 核心功能
1. **AI量表生成** — 输入描述自动生成专业量表
2. **在线施测** — 学生端测评作答
3. **AI报告生成** — 自动生成个性化分析报告
4. **AI异常检测** — 智能识别风险信号
5. **预警系统** — 分级预警通知

### 价值主张
> "让心理老师从重复性工作中解放出来，专注于真正需要人工干预的学生"

## 五、技术架构

### 技术栈

| 层级 | 技术 | 版本 |
|------|------|------|
| 后端 | Spring Boot | 3.5.5 |
| 框架 | JeecgBoot | 3.9.2 |
| ORM | MyBatis-Plus | 3.5.12 |
| 语言 | Java | 17 |
| 前端 | Vue | 3.x |
| UI | Ant Design Vue | 4.x |
| 构建 | Vite | 6.x |
| 数据库 | MySQL | 8.0 |
| 认证 | Apache Shiro + JWT | 2.0.5 |
| AI | LangChain4j (JeecgBoot airag) | - |

### 模块架构（AI优先）

```
jeecg-boot/jeecg-boot-module/jeecg-module-psyassess/
└── src/main/java/org/jeecg/modules/psyassess/
    ├── ai/             # 【核心】AI服务层（优先级最高）
    │   ├── ScaleAiService.java      # AI量表生成
    │   ├── ReportAiService.java     # AI报告生成
    │   ├── AnomalyAiService.java    # AI异常检测
    │   └── AIChatHandler.java       # AI调用封装
    ├── scale/          # 量表中心
    ├── testing/        # 施测中心
    ├── profile/        # 学生档案中心
    ├── report/         # 报告中心
    ├── alert/          # 预警中心
    └── common/         # 枚举、常量、VO、工具类
```

### 前端结构

```
jeecgboot-vue3/src/views/psyassess/
├── ai/               # 【核心】AI功能（优先开发）
│   ├── scale-gen.vue    # AI生成量表
│   ├── report-gen.vue   # AI生成报告预览
│   └── anomaly-detect.vue # AI异常检测结果
├── scale/            # 量表管理
├── testing/          # 施测中心
├── profile/          # 学生档案
├── report/           # 报告中心
├── alert/            # 预警中心
└── components/       # 共享组件
    ├── ScaleRenderer.vue
    ├── ScoreChart.vue
    ├── AlertBadge.vue
    └── ReportHtml.vue
```

## 六、数据库设计（AI优先优化）

### AI服务表（新增/强化）

| 表名 | 说明 | 关键字段 |
|------|------|---------|
| `psy_ai_model` | AI模型配置 | model_id, name, provider(OPENAI/DEEPSEEK/LOCAL), api_key(加密), params(JSON), is_default |
| `psy_ai_generation_log` | AI生成追溯 | gen_type(SCALE_GEN/REPORT_GEN/ANOMALY), model_id, input_prompt, output_content, token_used, latency_ms, is_accepted, feedback |
| `psy_ai_finetune_data` | 微调数据集 | data_type(SCALE/REPORT), content, label, used_count |

### 量表中心

| 表名 | 说明 | 关键字段 |
|------|------|---------|
| `psy_scale` | 量表主表 | code, name, category, source(BUILTIN/CUSTOM/AI_GENERATED), ai_model_id, version, tenant_id |
| `psy_scale_dimension` | 维度/因子 | scale_id, code, name, scoring_formula, severity_levels(JSON) |
| `psy_scale_question` | 题目 | scale_id, dimension_id, question_no, question_type, stem, is_reverse, ai_suggestion |
| `psy_scale_option` | 选项 | question_id, option_label, option_text, score |
| `psy_scale_rule` | 计分规则 | scale_id, rule_type, rule_config(JSON) |

### 施测中心

| 表名 | 说明 | 关键字段 |
|------|------|---------|
| `psy_test_task` | 施测任务 | scale_id, task_type, start_time, end_time, anonymity, target_config(JSON), status |
| `psy_test_record` | 作答记录 | task_id, student_id, status, duration_sec, ai_flags(JSON) |
| `psy_test_answer` | 答案明细 | record_id, question_id, question_no, answer_value, raw_score |
| `psy_test_result` | 评分结果 | record_id, total_score, standard_score, dimension_scores(JSON), severity_level, alert_triggered, ai_analysis |

### 预警中心

| 表名 | 说明 | 关键字段 |
|------|------|---------|
| `psy_alert_rule` | 预警规则 | alert_level, rule_type(SCORE_THRESHOLD/KEY_ITEM/SPIKE/AI_ANOMALY/BEHAVIOR), rule_config(JSON), notify_roles, cooldown_hours, ai_enabled |
| `psy_alert_record` | 预警记录 | rule_id, student_id, alert_level, trigger_data(JSON), ai_analysis, is_read |
| `psy_alert_handle` | 预警处理 | alert_id, handle_type, handle_comment, handle_result |

### 学生档案

| 表名 | 说明 | 关键字段 |
|------|------|---------|
| `psy_student` | 学生扩展信息 | user_id, student_no, school_id, grade_code, class_name, guardian_name |
| `psy_family_info` | 家庭信息 | student_id, member_role, parenting_style, marital_status |
| `psy_counsel_record` | 心理辅导记录 | student_id, teacher_id, counsel_date, assessment, intervention, outcome |

### 报告中心

| 表名 | 说明 | 关键字段 |
|------|------|---------|
| `psy_report_template` | 报告模板 | report_type, template_content, ai_prompt, ai_model_id |
| `psy_report` | 报告记录 | report_type, target_id, test_record_id, report_data(JSON), report_content(HTML), ai_generated, ai_model_id |

## 七、核心业务逻辑（AI优先）

### 1. AI量表生成引擎

```
用户输入量表描述 → AI生成JSON结构 → 人工审核确认 → 入库
    ↓
系统提示词(心理测评专家) → completions() → JSON解析 → 标记AI_GENERATED
```

**关键设计：**
- 支持多种量表类型：心理健康筛查、情绪状态、学习压力、社交适应
- AI生成后必须人工审核才能发布
- 保留AI生成日志便于追溯和反馈

### 2. AI报告生成引擎

```
测评结果 → 学生画像 → AI生成报告 → HTML渲染 → 存储/下载
    ↓
上下文组装(学生信息+得分+历史记录) → chat()流式生成 → 安全过滤 → 存储
```

**报告类型：**
| 类型 | 受众 | 内容重点 |
|------|------|----------|
| STUDENT | 学生 | 个人成长建议、正向引导 |
| PARENT | 家长 | 家庭沟通建议、观察要点 |
| TEACHER | 班主任 | 班级管理建议、关注重点 |
| SCHOOL | 学校 | 整体趋势分析、干预建议 |
| DISTRICT | 教育局 | 区域统计、资源配置建议 |

### 3. AI异常检测引擎

```
答题数据 → AI分析 → 风险识别 → 预警触发
    ↓
答题模式分析 + 关键词检测 + 历史对比 → completions() → 更新ai_analysis字段
```

**检测维度：**
- 答题一致性（矛盾回答检测）
- 极端选项偏好
- 开放式回答关键词分析
- 得分突增/突降
- 与常模偏差分析

### 4. 智能预警引擎

```
评分完成 → AI异常检测 → 规则匹配 → 预警生成 → 通知推送
    ↓
自动触发（同步） + AI深度分析（异步）
```

**三级预警体系：**

| 级别 | 触发条件 | 通知对象 | 响应时限 |
|------|----------|----------|----------|
| 轻度 | 单维度超常 | 班主任 | 24小时 |
| 中度 | 多维度异常 | 心理老师 + 班主任 | 12小时 |
| 严重 | 自伤/自杀关键项 | 心理老师 + 校领导 + 家长 | 1小时 |

## 八、API设计（AI优先）

### AI服务接口（核心）

| API路径 | 方法 | 说明 |
|---------|------|------|
| `/psyassess/ai/model/list` | GET | 获取AI模型列表 |
| `/psyassess/ai/model/save` | POST | 保存AI模型配置 |
| `/psyassess/ai/scale/generate` | POST | AI生成量表 |
| `/psyassess/ai/scale/preview` | POST | 预览AI生成结果 |
| `/psyassess/ai/report/generate` | POST | AI生成报告 |
| `/psyassess/ai/anomaly/detect` | POST | AI异常检测 |
| `/psyassess/ai/log/list` | GET | 获取AI生成日志 |
| `/psyassess/ai/log/feedback` | POST | 提交AI反馈 |

### 其他接口

| 模块 | 路径前缀 | 说明 |
|------|----------|------|
| 量表中心 | `/psyassess/scale` | CRUD + 发布/禁用 |
| 施测中心 | `/psyassess/testing` | 任务管理 + 作答流程 |
| 学生档案 | `/psyassess/profile` | 学生 + 家庭 + 辅导记录 |
| 报告中心 | `/psyassess/report` | 模板 + 报告生成/查看 |
| 预警中心 | `/psyassess/alert` | 规则 + 记录 + 处理 |

## 九、前端页面清单（AI优先顺序）

| 优先级 | 页面 | 说明 |
|--------|------|------|
| P0 | AI量表生成 | 输入描述生成量表 |
| P0 | AI报告预览 | 预览AI生成的报告 |
| P1 | 量表列表/管理 | 量表CRUD |
| P1 | 施测任务管理 | 创建/编辑任务 |
| P1 | 学生作答页 | 在线测评界面 |
| P2 | AI异常检测日志 | 查看AI检测结果 |
| P2 | 预警记录 | 预警列表和处理 |
| P2 | 学生档案 | 学生信息管理 |
| P3 | 报告中心 | 报告查看和下载 |
| P3 | AI使用统计 | AI调用统计面板 |

## 十、部署与运维

### 部署架构

```
┌─────────────────────────────────────────────────────┐
│                    Nginx                            │
├─────────────────────────────────────────────────────┤
│              Spring Boot Application                │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐  │
│  │  Web层      │ │  Service层  │ │  Repository │  │
│  └─────────────┘ └─────────────┘ └─────────────┘  │
├─────────────────────────────────────────────────────┤
│                    MySQL 8.0                        │
└─────────────────────────────────────────────────────┘
```

### 配置要求

| 环境 | CPU | 内存 | 存储 |
|------|-----|------|------|
| 开发 | 2核 | 4GB | 50GB |
| 测试 | 4核 | 8GB | 100GB |
| 生产 | 8核 | 16GB | 200GB+ |

### 安全考虑

1. **数据加密**：敏感数据（学生信息、测评结果）加密存储
2. **访问控制**：基于角色的细粒度权限控制
3. **审计日志**：记录所有操作和AI调用
4. **XSS防护**：报告HTML内容安全过滤
5. **API限流**：防止恶意调用

## 十一、成功标准

### 业务指标
- 心理老师报告撰写时间减少 80%
- 预警响应时间从平均 48 小时缩短到 2 小时
- 量表创建效率提升 5 倍（AI生成）

### 技术指标
- API响应时间 < 200ms（非AI调用）
- AI生成报告时间 < 30秒
- 系统可用性 > 99.5%

## 十二、下一步行动

1. **验证AI能力**：测试AI量表生成和报告生成的质量
2. **用户验证**：邀请5-10位心理老师试用核心功能
3. **技术验证**：完成核心模块的技术原型
4. **商业验证**：与1-2所学校签署试点协议

## 十三、CEO审查修正（plan-ceo-review 2026-05-18）

> 以下内容由 SCOPE EXPANSION 模式审查产生。全部11个审查章节已通过，10个发现已接受并纳入计划。

### 13.1 测试策略

完整的测试矩阵，按优先级分层：

| 层 | 范围 | 占比目标 |
|----|------|----------|
| 单元测试 | 所有Service方法、JSON解析器、计分引擎、预警规则匹配 | 70% |
| 集成测试 | AI调用Mock链路、事件驱动链路、消息通知链路、作答流程 | 20% |
| E2E测试 | 学生作答完整流程、预警处理完整流程 | 10% |

**AI质量评测集：**
- 量表生成评测：20个典型量表描述 → 人工评分（结构完整性、题目合理性、计分正确性）
- 报告生成评测：10份已知测评结果 → 人工评分（可读性、专业性、无幻觉）
- 异常检测评测：标注数据集（已知风险/正常），测量准确率和召回率

**关键边界用例：**
- AI输入为空/纯空格/超长/特殊字符
- 作答localStorage保存+恢复、幂等性token防重复提交、断线重连
- 预警去重（同学生/同时段/同类型）、超时升级链接完整性
- 批量报告部分失败（50人中5人失败→其余45人正常完成并标记失败项）

### 13.1.1 测试缺口补充（ENG REVIEW 2026-05-18）

以下16个测试缺口已全部接受，纳入测试计划：

**CRITICAL (G1-G3)：**

| ID | 缺口 | 测试目标 | 前置条件 | 关键断言 | 优先级 |
|----|------|----------|----------|----------|--------|
| G1 | 事件驱动异步失败 | 验证异步AI分析异常后预警状态正确 | Mock AI抛异常 | alert_record不被标记为"已分析"；重试触发；异常日志记录 | P0 |
| G2 | 租户数据隔离 | 验证跨租户数据不可访问 | 租户A/B各有一条psy数据 | 租户A查询返回仅自身数据；租户B直接访问租户A的ID返回404/空 | P0 |
| G3 | 预警冷却期去重 | 验证同规则冷却期内不重复告警 | cooldown_hours=2，间隔1小时触发2次 | 仅生成1条alert_record；第2次触发日志记录"冷却中跳过" | P0 |

**HIGH (G4-G8)：**

| ID | 缺口 | 测试目标 | 前置条件 | 关键断言 | 优先级 |
|----|------|----------|----------|----------|--------|
| G4 | AI超时降级路径 | 验证AI超时后规则引擎降级结果正确 | Mock AI延迟>10s | 返回规则引擎评分结果；response含degraded=true标记；不抛500 | P1 |
| G5 | Shiro权限边界 | 验证5种角色各自端点访问权限 | 配置filter chain | 未认证→401；学生角色→可访问/testing不可访问/alert/handle；管理员→全权限 | P1 |
| G6 | Feature Flag开关 | 验证psyassess.enabled=false行为 | application.yml配置 | 所有/psyassess/**返回503+JSON错误体；动态切换后即时生效 | P1 |
| G7 | PIPL合规完整链路 | 验证consent状态机+级联删除+数据导出 | 插入完整学生数据 | 待签署→已签署→已撤回状态流转正确；级联删除后所有关联表无残留；导出JSON含所有子表 | P1 |
| G8 | 通知多通道送达 | 验证SMS/微信/WebSocket三种通道 | Mock SMS/微信网关 | WebSocket消息体含alert_level+student_name；SMS失败记录到retry_queue；微信不可达降级日志 | P1 |

**MEDIUM (G9-G14)：**

| ID | 缺口 | 测试目标 | 前置条件 | 关键断言 | 优先级 |
|----|------|----------|----------|----------|--------|
| G9 | 流式报告中断恢复 | 验证chat() SSE中断后前端状态正确 | 断网模拟 | SSE重连后从断点继续；已渲染内容不丢失；超时30s后提示降级 | P2 |
| G10 | 批量处理并发压力 | 验证100并发+线程池耗尽行为 | 线程池core=10,max=10 | 进度回调每完成1条推送；线程池满时任务排队不丢失 | P2 |
| G11 | 5种报告类型差异化 | 验证每种受众收到不同内容重点 | 同一测评结果生成5种报告 | STUDENT含正向引导词；PARENT含沟通建议；SCHOOL含趋势分析 | P2 |
| G12 | DB迁移回滚 | 验证迁移可逆且数据不丢失 | 正向迁移完成+插入测试数据 | Reverse后表不存在；再Forward后数据恢复 | P2 |
| G13 | 量表预览不持久化 | 验证预览后数据库中无残留 | 调用预览API | psy_scale/psy_scale_question/psy_scale_option三表均无新增 | P2 |
| G14 | 预警升级通知链路 | 验证超时升级后新通知对象收到消息 | 创建轻度预警，模拟超时 | 升级后校领导/家长收到通知；原通知对象不重复收到 | P2 |

**LOW (G15-G16)：**

| ID | 缺口 | 测试目标 | 前置条件 | 关键断言 | 优先级 |
|----|------|----------|----------|----------|--------|
| G15 | JimuBI大屏集成 | 验证数据读取器输出正确 | JimuBI环境就绪 | 数据集查询结果与直接SQL一致；权限模型不冲突 | P3 |
| G16 | 学期报告自动触发 | 验证Cron+学期边界正确 | 配置Cron表达式 | 学期最后一天触发；跨学期不触发；报告生成成功 | P3 |

### 13.1.2 代码质量偏离修正（ENG REVIEW 2026-05-18）

以下 8 项偏离 JeecgBoot 现有代码惯例的发现已全部接受，纳入实现规范：

**强制修改（F1-F4）：**

| ID | 偏离 | 当前 Plan | 修正后 | 依据 |
|----|------|-----------|--------|------|
| F1 | AIChatHandler 引用歧义 | `psyassess/ai/AIChatHandler.java`（自建） | 通过 Maven 依赖引入 `jeecg-boot-module-airag`，import `org.jeecg.modules.airag.llm.handler.AIChatHandler` | 该类已存在于 airag 模块，无需复制 |
| F2 | Service 命名偏离惯例 | `ScaleAiService` / `ReportAiService` | 改为 `IScaleAiService` 接口 + `ScaleAiServiceImpl` 实现 | Jeecg 全局惯例：I 前缀区分接口 |
| F3 | 缺少 @AutoLog 审计 | 未声明 | AI 调用、报告生成、预警触发、量表发布、consent 变更等关键操作均加 `@AutoLog(value="...", operateType=...)` | [JeecgDemoController](file:///Users/suzhenyu/trae/JeecgBoot/jeecg-boot/jeecg-boot-module/jeecg-module-demo/src/main/java/org/jeecg/modules/demo/test/controller/JeecgDemoController.java) 已有完善审计体系 |
| F4 | 缺少 @Transactional 边界 | 未声明 | 同步阶段（作答→评分→同步预警规则匹配）在一个事务内；异步阶段（AI 分析→写回 ai_analysis）独立事务 | `ServiceImpl` 基类已支持 `@Transactional(rollbackFor = Exception.class)` |

**建议补充（F5-F8）：**

| ID | 偏离 | 补充规范 |
|----|------|----------|
| F5 | 缺少 @PermissionData 数据权限 | 在 Controller 分页查询方法上加 `@PermissionData(pageComponent = "psyassess/...")`；心理老师→任教班级，班主任→本班，管理员→全校 |
| F6 | 缺少 Bean Validation | API 入参实体加 `@NotBlank`/`@NotNull`/`@Size`；Controller 方法参数加 `@Valid`；触发 `MethodArgumentNotValidException` → 全局异常处理器自动返回 400 |
| F7 | XSS 过滤机制未指定 | AI 生成 HTML 报告入库前经 `Jsoup.clean(html, Safelist.basic())` 白名单过滤；仅允许 p/br/b/i/ul/li/h3/h4/table 等安全标签 |
| F8 | Redis 缓存策略不完整 | 复用 Jeecg 现有 `@Cacheable` + Redis TTL 基础设施：报告缓存 `cacheNames = "psyassess:report#3600"`，量表缓存 `cacheNames = "psyassess:scale#1800"` |

### 13.2 AI调用性能策略

| 配置项 | 值 | 说明 |
|--------|-----|------|
| completions()超时 | 10秒 | 量表生成/异常检测 |
| chat()超时 | 30秒 | 报告生成流式 |
| 最大重试 | 2次 | 指数退避: 1s→4s |
| 并发AI调用上限 | 10 | Semaphore信号量控制 |
| 降级策略 | 返回规则引擎结果 | AI超时时使用规则评分替代AI分析 |
| 报告缓存 | 同类报告1小时复用 | 避免重复生成 |

### 13.2.1 性能缺口补充（ENG REVIEW 2026-05-18）

以下 8 项性能缺口已全部接受，纳入实现规范：

| ID | 缺口 | 严重度 | 影响 | 解决方案 |
|----|------|--------|------|----------|
| P1 | N+1 量表加载 | CRITICAL | 50题量表产生约255次SQL | MyBatis XML 用 LEFT JOIN 一次查询完整量表树（scale→dimensions→questions→options）；或 mapper 加 `@ResultMap` 嵌套映射 |
| P2 | 仪表盘全表聚合 | HIGH | 全校预警统计扫描全表 | 凌晨定时任务生成 `psy_daily_stats` 汇总表；仪表盘只读汇总表；利用 Quartz 现有基础设施 |
| P3 | 慢SQL阈值不匹配 | HIGH | Druid 5000ms 告警，但 Plan 要求 API <200ms | 新增 200ms+ 的应用层 warn 日志；保留 Druid 5s 作为兜底 |
| P4 | 批量写入批次大小 | MEDIUM | saveBatch 默认 1000条/批 | 答案写入：`saveBatch(list, 50)` 批大小；报告生成分页获取 |
| P5 | 纵向追踪缺少复合索引 | MEDIUM | 缺失 `psy_test_result(student_id, created_at)` | 补充至 13.3 索引计划（第9个索引） |
| P6 | 批量报告内存压力 | MEDIUM | 1000人×5KB ≈ 5MB 可控但不优雅 | 报告生成流式写入 S3/MinIO，不在堆内存缓存全部 HTML |
| P7 | Redis 单点 | LOW | 生产 Redis 单机宕机→缓存穿透 | 非阻塞项，后续迭代部署 Redis Sentinel |
| P8 | 生产慢SQL阈值宽松 | LOW | Druid 5s 对企业应用偏宽松 | 生产环境建议调为 1000ms |

**现有基础设施基线：** Druid maxActive=1000，PSCache=20/连接，Redis 单机，Tomcat compression=enabled，Quartz 10线程，文件上传限制 10MB。

### 13.3 数据库索引计划

| 表 | 索引 | 类型 |
|----|------|------|
| `psy_test_record` | (student_id, created_at) | 联合索引 |
| `psy_test_record` | (task_id, status) | 联合索引 |
| `psy_test_answer` | (record_id, question_no) | 联合索引 |
| `psy_alert_record` | (student_id, is_read, created_at) | 联合索引 |
| `psy_alert_record` | (alert_level, created_at) | 联合索引 |
| `psy_scale_question` | (scale_id, question_no) | 联合索引 |
| `psy_test_task` | (status, end_time) | 联合索引 |
| `psy_report` | (target_id, report_type) | 联合索引 |
| `psy_test_result` | (student_id, created_at) | 联合索引（纵向追踪趋势分析） |

**租户隔离（CRITICAL — 外部评审发现）：**
全部20张psy_*表必须添加`tenant_id`列（当前仅psy_scale有），并在`MybatisPlusSaasConfig.TENANT_TABLE`中注册。缺失一张表=该表数据跨租户泄露。

### 13.4 批量处理策略

- 线程池并发执行，并发度=min(CPU核数, 10)
- 每批完成回调通知前端（进度条接口：已完成数/总数）
- 部分失败隔离：失败的记录单独标记，不影响成功记录
- 进程级去重：同一触发源30秒内不重复创建批次

### 13.5 可观测性方案

**关键指标：**
| 指标 | 告警阈值 |
|------|----------|
| AI调用成功率 | < 90% 告警 |
| 预警处理率（已处理/总数） | < 70% 告警 |
| 报告生成p99延迟 | > 60秒 告警 |
| 消息队列深度 | > 1000 告警 |

- 健康检查端点：`/actuator/health` 包含AI服务连通性
- 系统事件日志：评分完成、预警触发、通知发送 —— 结构化日志
- 与JeecgBoot现有监控体系的集成点：复用Spring Boot Actuator

### 13.6 部署与回滚方案

**Feature Flag:** `psyassess.enabled=true` 总开关，关闭时所有psyassess端点返回503

**部署顺序：**
1. DB迁移（正向SQL：建表+索引）→ 验证
2. 模块注册（jeecg-module-psyassess加入pom + 模块扫描配置）
3. TENANT_TABLE配置（全部20张psy_*表）
4. 应用部署 → 冒烟测试

**5分钟验证清单：**
- [ ] `/actuator/health` → UP
- [ ] `/psyassess/ai/model/list` → 200 OK
- [ ] Feature Flag确认 psyassess.enabled=true

**1小时验证清单：**
- [ ] AI模型配置连接测试通过
- [ ] 量表CRUD正常
- [ ] 学生作答页访问正常

**回滚步骤：**
1. `git revert <commit>` → 部署旧代码
2. DB迁移reverse脚本（只DROP psy_*表，保留数据备份）
3. 关闭Feature Flag: `psyassess.enabled=false`

### 13.7 UX设计规范

**交互状态矩阵（全覆盖）：**

| 页面 | Loading | Empty | Error | Success | Partial |
|------|---------|-------|-------|---------|---------|
| AI量表生成 | 骨架屏 + 生成步骤提示 | 引导输入首张量表 | 生成失败+重试按钮+原因说明 | 预览+确认/驳回 | — |
| AI报告生成 | 分段提示("分析答题模式..."→"生成建议...")流式输出 | — | 生成失败+降级为规则报告 | HTML渲染+下载 | — |
| 学生作答页 | 逐题加载 | — | 网络断开提示+自动重连+本地恢复 | 提交成功+简要反馈 | 未完成保存恢复提示 |
| 预警面板 | 列表加载 | "暂无预警"插图 | 加载失败+刷新 | 列表+筛选 | — |
| 仪表盘 | 数据加载动画 | 各面板空状态 | 面板级Error+刷新 | 全部渲染 | 部分面板Error其余正常 |

**AI等待体验：** chat()流式输出→逐字渲染，不可仅转圈。分阶段展示当前AI在做什么。

### 13.8 PIPL合规（个人信息保护法）

- `psy_family_info` 增加 `consent_status`（待签署/已签署/已撤回）+ `consent_date`
- 监护人同意管理接口：`/psyassess/profile/consent`
- 数据留存策略：学生毕业后自动匿名化（保留统计级别数据，清除个人标识）
- 删除权：`/psyassess/profile/{studentId}/delete` 级联删除所有关联数据
- 数据可携带权：`/psyassess/profile/{studentId}/export` JSON格式导出

### 13.9 平台演进路线

| Phase | 内容 | 依赖当前设计 |
|-------|------|-------------|
| Phase 1 | 6模块MVP + 8项扩展（当前计划） | — |
| Phase 2 | AI危机干预聊天机器人、家长端App | 学生档案 + AI服务层 |
| Phase 3 | 跨学校区域心理健康大数据分析 | 仪表盘 + 纵向追踪 + psy_ai_finetune_data |

**数据壁垒：** `psy_ai_finetune_data` 表将积累标注数据用于未来微调专用心理评估模型。

### 13.10 事件驱动基础设施标注

⚠️ **事件驱动层为新增基础设施，需从零构建。** JeecgBoot全库零个@EventListener、零个自定义事件。需要：
- 事件类定义（PsyScoreCompleteEvent等）
- ApplicationEventPublisher注入
- @EventListener/@TransactionalEventListener监听器
- 统一异常处理器（非Spring默认吞错）
- 需在`@EnableAsync`配置中启用异步事件处理

### 13.11 JimuBI集成实际依赖

⚠️ **JimuBI大屏不是"M"工作量，调整为"L"（较大）。** 额外依赖：
- Redis（部署图中需补充）
- 独立DB初始化SQL
- 每个大屏数据集需编写自定义数据读取器
- 权限模型与Shiro不同（需独立处理）

### 13.12 通知基础设施规划

当前JeecgBoot有WebSocket + Email + SysAnnouncement。需新增：
- SMS网关对接（家长通知场景）
- 微信企业号/公众号对接（家长通知场景）
- 模板化消息引擎（不同预警级别→不同通知模板）

### 13.13 外部安全审查补充（Outside Voice 2026-05-18）

以下 8 项独立安全审查发现已全部接受，纳入计划：

**CRITICAL（O1-O3）——必须修正：**

| ID | 风险 | 问题 | 修正方案 |
|----|------|------|----------|
| O1 | 字段级加密缺失 | `psy_ai_model.api_key` / `psy_family_info` / `psy_test_result.dimension_scores` 裸存 | 引入 AES-256-GCM 字段级加密；MyBatis-Plus TypeHandler 层透明加解密；密钥通过环境变量 `JEECG_DATA_ENCRYPT_KEY` 注入，不硬编码 |
| O2 | 异步线程上下文丢失 | 事件驱动中 TenantContext/SecurityUtils 是 ThreadLocal，异步线程中断裂，tenant_id 回退 "0" | 事件对象显式携带 `tenant_id`/`user_id`；异步监听器入口 `TenantContext.setTenant()` + finally 清理；异步线程池用 `TaskDecorator` 自动传播上下文 |
| O3 | AI 量表无心理测量学效度 | AI 生成量表无信效度验证，可能产生无效测评工具 | `psy_scale` 增加 `psychometric_status`（DRAFT/PILOT_TESTED/VALIDATED/NORMED）；AI 生成量表标记 DRAFT → 需 ≥2 名持证心理专业人员审核；prompt 禁止生成含建议/引导性描述 |

**HIGH（O4-O7）——强烈建议：**

| ID | 风险 | 修正方案 |
|----|------|----------|
| O4 | 教育行业合规缺失 | 纳入等保三级技术要求清单；`psy_consent` 独立成表含监护人双因子确认；`psy_student` 增加 `annual_screening_status` 支持教育部"每学年至少一次全员测评" |
| O5 | 多租户隔离隐蔽死角 | 禁止 psyassess 使用 `@Select` 原生 SQL / `JdbcTemplate`；`tenant_id` 为空时抛异常而非回退 `"0"`；JOIN 查询逐表验证租户过滤 |
| O6 | 学校运维能力不匹配 | Docker Compose 一键部署；DB 迁移预部署独立执行 + 可视化验证页；AI 服务 circuit breaker（3次失败→降级规则引擎） |
| O7 | AI 报告无未成年人安全护栏 | 内容安全审核层：敏感关键词扫描后隐藏学生侧展示；STUDENT 报告不展示原始得分/临床诊断/统计数据；建立 `psy_report_safety_log` 表 |

**MED-HIGH（O8）：**

| ID | 风险 | 修正方案 |
|----|------|----------|
| O8 | 预警通知隐私泄露 | SMS 消息体仅发通用通知不含病情；5分钟窗口内聚合为摘要；`psy_family_info` 增加 `notify_preference`（含双亲/单方/禁止）；严重预警增加电话录音备份通道 |

---

## GSTACK REVIEW REPORT

| Review | Trigger | Why | Runs | Status | Findings |
|--------|---------|-----|------|--------|----------|
| CEO Review | `/plan-ceo-review` | Scope & strategy — SCOPE EXPANSION | 1 | COMPLETE | 10 findings: 10 accepted, 0 deferred — 2 CRITICAL GAPS in Error Registry |
| Eng Review | `/plan-eng-review` | Architecture, tests, code quality, performance | 1 | COMPLETE | 34 findings across 4 sections: all accepted |
| Codex Review | — | Independent 2nd opinion | — | N/A | — |
| Design Review | — | UI/UX gaps | — | PENDING | — |
| DX Review | — | Developer experience gaps | — | PENDING | — |

### Eng Review Summary (2026-05-18)

| Section | Findings | Status |
|---------|----------|--------|
| **Architecture** | 2 findings: module dependency (AIChatHandler ref) + Shiro filter chain (all ACCEPTED) | ✅ |
| **Test Coverage** | 16 GAPs: 3 CRITICAL + 5 HIGH + 6 MEDIUM + 2 LOW (all ACCEPTED, written to 13.1.1) | ✅ |
| **Code Quality** | 8 deviations: 4 mandatory + 4 advisory (all ACCEPTED, written to 13.1.2) | ✅ |
| **Performance** | 8 findings: 1 CRITICAL + 2 HIGH + 3 MEDIUM + 2 LOW (all ACCEPTED, written to 13.2.1) | ✅ |
| **Outside Voice** | 8 findings: 3 CRITICAL + 4 HIGH + 1 MED-HIGH (all ACCEPTED, written to 13.13) | ✅ |
| **Total** | **42 findings, 42 accepted, 0 deferred, 0 rejected** | |

**CRITICAL findings requiring immediate attention (before any code is written):**

| # | Finding | Section | Root Cause |
|---|---------|---------|-------------|
| C1 | N+1 量表加载 — 50题量表产生 ~255 次 SQL | Performance P1 | 无关联查询策略 |
| C2 | 字段级加密缺失 — API Key + 心理数据裸存 | Outside O1 | TypeHandler 加密方案缺失 |
| C3 | 异步线程上下文丢失 — 跨租户数据泄露 | Outside O2 | ThreadLocal + 事件驱动冲突 |
| C4 | AI 量表无心理测量学效度 | Outside O3 | 缺 psychometric_status 字段 + 审核流程 |
| C5 | 事件驱动异步失败 — 预警状态不正确 | Test G1 | 异步异常处理缺失 |
| C6 | 租户数据隔离 — 跨租户访问未设防 | Test G2 | tenant_id 注册 + 权限验证 |
| C7 | 预警冷却期去重 — 重复告警 | Test G3 | cooldown_hours 逻辑未测试 |

**Plan health score: 72/100** (baseline 65 → +7 from CEO review corrections → +5 from Eng review fixes → current 72)

**VERDICT:** Eng Review complete. 42 findings accepted, 7 CRITICAL items must be resolved before implementation begins. Design Review and DX Review recommended as follow-ups given the UI-heavy scope (14 Vue pages + JimuBI dashboard).
