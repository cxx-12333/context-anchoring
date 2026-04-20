# <PROJECT_NAME> - 决策记录

> 代码记录结果，不记录推理。此文档持久化决策背后的推理。
> 参考：https://martinfowler.com/articles/reduce-friction-ai/context-anchoring.html

## 决策

### D001: 采用 Context Anchoring 协作框架

- **日期**: <DATE>
- **决策**: 采用两层上下文 + 四层协作架构
- **理由**: AI 上下文窗口有限，推理比决策先消失。将决策上下文外化为文档，作为跨会话权威参考。
- **被拒绝的替代方案**:
  - 仅 AGENTS.md：无法持久化特性级决策和推理
  - 仅 Memory：不稳定，跨工具不可用，内容不可审计
  - 不使用文档：推理只在对话中，会话结束即丢失
- **影响范围**: 全局协作流程
- **状态**: ✅ 已确认
