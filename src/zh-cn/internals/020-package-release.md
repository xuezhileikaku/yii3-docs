# 020 — 包发布

## 标准

- 没有关键问题。
- 公共 API 更改不太可能。在没有报告可能需要 API 更改的问题的情况下过去了一段时间。
- 所有依赖项都是稳定的。
- 接近 100% 的测试覆盖率,理想情况下为 100% 的 MSI 分数。
- README 可以。
- 除了特殊情况外,所有内容都有类型提示。
- Psalm 分析至少在级别 2 上通过。
- phpdoc 可以。
- 公共 API 可以。

## 发布说明

通过 [Yii 开发工具](005-development-tool.md)发布包。

1. 检查您是否可以在本地签名提交(参见[签名提交](https://docs.github.com/en/authentication/managing-commit-signature-verification/signing-commits))。

2. 从 `master` 分支拉取最后的更改:

```shell
./yii-dev git/checkout master package-name
./yii-dev git/pull package-name
```

3. 检查包是否符合上述标准。

4. 运行 `release/make` 命令:

```shell
./yii-dev release/make package-name
```

5. 选择版本类型(主要、次要或补丁)。

6. 在问题"推送提交和标签,并在 GitHub 上发布?"上,检查差异。如果差异可以,回答"yes"。

7. 对于主要和次要发布,在 [Yii Framework News](https://www.yiiframework.com/news) 上添加带有发布说明的记录。
