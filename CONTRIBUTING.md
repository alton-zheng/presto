## 如何贡献项目

领取或创建新的 [Issue](https://github.com/alton-zheng/spark-cn/issues)，如 [issue 1](https://github.com/alton-zheng/spark-cn/issues/1) ，在编辑栏的右侧添加自己为 `issue`。

在 [GitHub](https://github.com/alton-zheng/spark-cn/fork) 上 `fork` 项目到自己的仓库，你的如 `user_name/presto-doc`，然后 `clone` 到本地，并设置用户信息。

```bash
$ git clone https://github.com/alton-zheng/presto-doc.git

$ cd presto-doc
```

修改代码后提交，并推送到自己的仓库，注意修改提交消息为对应 Issue 号和描述。

```bash
# 更新内容

$ git commit -a -s

# 在提交对话框里添加类似如下内容 "Fix issue #1: 描述你的修改内容"

$ git push
```

在 [GitHub](https://github.com/alton-zheng/spark-cn/pulls) 上提交 `Pull Request`，添加标签，并邀请维护者进行 `Review`。

定期使用源项目仓库内容同步更新自己仓库的内容。

```bash
$ git remote add upstream https://github.com/alton-zheng/presto-doc

$ git fetch upstream

$ git rebase upstream/master

$ git push -f origin master
```

## 排版规范

本开源书籍排版布局和翻译风格上参考**阮一峰**老师的 [中文技术文档的写作规范](https://github.com/ruanyf/document-style-guide)







