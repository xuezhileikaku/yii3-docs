# 013 — 代码审查

代码审查对于项目的成功至关重要,与贡献代码一样重要。

审查通过 GitHub 拉取请求处理。当请求准备好审查时,"status: code review" 标签会添加到其中。

[GitHub 上提供了需要审查的完整拉取请求列表](https://github.com/search?q=org%3Ayiisoft+label%3A"status%3Acode+review"&state=open&type=Issues)。

## 指南

- 检出拉取请求分支,在 IDE 中打开项目,了解大局。
- 拉取请求总体上是否有意义?
- 运行测试和/或使用来自拉取请求的代码的应用程序。这些可以吗?
- 阅读拉取请求中的所有代码行。
- 可以做得更简单吗?
- 是否有安全问题?
- 是否有性能问题?
- 留下评论时,要有礼貌。
- 避免代码格式化评论。

## 必需部分

- [ ] 没有代码会失败并且有代码会通过的测试。
- [ ] 类型提示。
- [ ] `declare(strict_types=1)`。
- [ ] 文档:phpdoc、yiisoft/docs。
- [ ] 变更日志条目(如果包是稳定的)。
