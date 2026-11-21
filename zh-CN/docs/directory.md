> **原文路径**：`docs/directory.md` | 本文档为中文学习翻译版，保留英文原文件以便对照。

# 应用目录打包包体（Directory packages）

仅使用 Python 标准库即可构建运行时文件夹与 ZIP 包：

```sh
python3 scripts/pack_plugin.py --target claude --output /tmp/egm-release
python3 scripts/pack_plugin.py --target openai --output /tmp/egm-release
```

输出结果包含 `egm-claude/` 与 `egm-claude.zip`，或 `egm-openai/` 与 `egm-openai.zip`。若输出路径已存在，脚本会拒绝覆盖。ZIP 内的条目顺序、时间戳和权限均已固定；在相同的 Python/zlib 运行时下，相同的输入会生成完全一致的归档包。

白名单包含技能指令与参考资料、专家角色、文档标准、运行时指南、清单文件及图标；排除网站、测试、工作流、插件市场目录、钩子（hooks）和构建工具。Claude 目标保留其原生智能体适配器；OpenAI 目标排除 Claude 组件。所有运行时文件均不具可执行权限。符号链接形式的运行时路径以及越界（escaping）的清单引用均会被拒绝。贡献者指南、AI 政策及 Markdown PR 模板均被排除，运行时文档中仅链接至它们的仓库副本。打包产物不包含任何工作流配置。运行时指南列表采用显式声明：新的宿主指南在完成打包要求评审前不会进入这些归档包。

切勿将生成的运行时目录或 ZIP 包提交到 `main` 分支。Claude 插件市场目录是从仓库根目录安装的，如果再多出一份副本，就会产生嵌套的 Claude 清单，而 Cowork 会拒绝加载嵌套清单。对于 Anthropic 的应用目录，其门户读取的是仓库地址、插件路径与跟踪分支，不接受上传 ZIP 包。提交前请按照 [Claude 发布分支分发方案](designs/claude_release_branch_distribution.md) 将运行时发布到自动生成的分支。
对于 OpenAI，请上传 `egm-openai.zip`。仅仅生成构建产物并不意味着已确立安装可行性、工作流支持或获得应用目录批准。
