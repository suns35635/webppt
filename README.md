# webppt
webppt

原始输入信息

网页ppt  牛逼啊 效果真好看 https://suns35635.github.io/webppt/


# PDF 智能问答功能 - 产品需求文档（PRD）

## 1. 背景与目标 (Context & Goals)
- **问题描述**：传统 PDF 工具无法基于选中文本提供智能解释或对话支持，用户需手动复制粘贴并切换上下文。
- **目标**：构建一个支持选中 PDF 内容后即时与 AI 对话的功能，响应速度 < 5s，正确率 > 95%，安全合规。
- **需求来源**：
  - 用户反馈：不懂专业文献、不便整理笔记。
  - 技术趋势：LLM 融合式阅读场景逐渐流行。
  - 内部技术研究：《AI 对话式网站构建综合研究报告》。

## 2. 用户故事 (User Stories)
- **As a** 研究人员，**I want to** 选中 PDF 中的复杂段落获得 AI 解读，**so that** 我能快速理解其含义。
- **As a** 产品经理，**I want to** 对整页内容进行摘要/生成导图，**so that** 我能快速把握整体结构。

## 3. 功能描述 (Functional Description)

### 3.1 正常流程
1. 用户打开 PDF，浏览页面。
2. 选中一段文本。
3. 系统显示浮动菜单：AI 聊天 / 翻译 / 解释。
4. 用户点击菜单项，系统准备上下文，构造 Prompt。
5. 调用 LLM API（前端 fetch + 用户 API key）。
6. 实时返回响应内容，展示在右侧面板中。

### 3.2 异常流程
- `[ERR-001]` 选中文本为空或过短：提示“请选择更完整的文本片段”。
- `[ERR-002]` API 请求失败：提示“AI 服务暂时不可用，请稍后重试”。
- `[ERR-003]` Token 超限：提示“选中文本过长，已自动截断”。

### 3.3 界面交互
- **组件**：@tato30/vue-pdf、floating-vue 悬浮菜单、右侧 AI 面板。
- **上下文菜单定位**：使用 `window.getSelection()` + `getBoundingClientRect()`。

## 4. 数据需求与埋点 (Data Requirements & Tracking)
- **记录字段**：
  - `action`: ai_request
  - `text_length`, `page_number`, `response_time`, `error_type`
- **埋点示例**：
```json
{
  "event": "ai_request",
  "text_length": 121,
  "provider": "openai",
  "mode": "explain",
  "result": "success"
}
```

## 5. 验收标准 (Acceptance Criteria)

| Given | When | Then |
|-------|------|------|
| 用户已输入有效 API 密钥 | 点击“解释”按钮 | 系统返回解释内容，5s 内显示 |
| 用户未选中文本 | 点击 AI 按钮 | 提示“请选择文本” |
| 用户选择整页 | 点击“生成导图” | 成功渲染思维导图（G6） |

## 6. 非功能性需求 (Non-Functional Requirements)
- 性能：95% 请求响应时间 < 3 秒（网络稳定时）
- 安全性：API 密钥仅存在内存，禁止写入 localStorage
- 可用性：AI 功能可用性 > 99.5%，异常自动重试1次

## 7. 发布计划与范围 (Release Plan & Scope)
- 发布版本：v1.0.0-beta
- 灰度策略：仅面向高级用户开启（用户需输入 API 密钥）
- 技术路线：纯前端实现，所有 AI 请求由用户浏览器完成

## 8. 未来扩展 (Future Considerations)
- 多语言翻译增强（支持法语、日语）
- AI 搜索整份 PDF
- 后端中转模式（解决密钥安全）

## 9. 风险管理与变更控制 (Risk Management & Change Control)

### 风险识别与缓解

| 风险类型 | 说明 | 缓解措施 |
|----------|------|-----------|
| 需求理解偏差 | 用户对 AI 对话预期与实际差距大 | 明确交互边界，设定提示引导，使用原型测试 |
| 技术实现难度 | LangChain.js 在纯前端中性能不稳定 | 引入异步 chunk 加载，按需索引构建 |
| 模型 API 变动 | OpenAI 或 DeepSeek 接口变动、限流 | 模块化封装 API 层，支持多模型切换 |
| 用户上传隐私数据 | PDF 中可能包含敏感数据 | 明确隐私协议，用户侧处理，提醒用户 |
| Token 超限 / 响应慢 | 长文本 + 上下文组合易超出 Token 限制 | 使用 RecursiveSplitter + Token 估算工具 |

### 变更控制流程

- 提出 → 评估 → 审批 → 实施与同步
- 所有变更需附原始链接、影响评估、版本更新说明

## 10. 度量与评估 (Metrics & Evaluation)

| 指标 | 目标值 | 说明 |
|------|--------|------|
| PRD 缺失率 | < 5% | 自动校验必填字段是否缺失 |
| Q&A 轮次 | ≤ 2 轮 | 开发测试阶段澄清频次 |
| 响应性能 | < 3s | AI 请求返回耗时 95 分位 |
| 满意度评分 | ≥ 80% | 用户交互后打分 |

