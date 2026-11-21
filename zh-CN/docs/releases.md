> **原文路径**：`docs/releases.md` | 本文档为中文学习翻译版，保留英文原文件以便对照。

# 版本化发布

权威版本号位于 `plugin.json` 的 `version` 字段。每一个声明了 EGM 版本号的宿主清单（manifest）都必须与之一致，包括实验性适配器（例如存在时的 Cursor 清单）。不含 `version` 字段的插件市场目录（marketplace catalog）无需添加版本号。版本号同步并不意味着将该宿主纳入正式发布支持范围；其他宿主适配器仍需经过显式打包与验证。

在 EGM 处于 1.0 之前的阶段，兼容性修复使用补丁版本递增（patch bump），新特性或不兼容的工作流变更使用次版本递增（minor bump），并在发布说明中解释破坏性变更。发布 1.0 版本要求技能名称、参数、输出路径和报告格式具备稳定契约。切勿在同一个标签（tag）下更改已发布的产物。

## 准备候选版本

1. 在一个经过评审的 PR 中，将 `plugin.json` 及所有宿主清单（`.claude-plugin/plugin.json`、`.codex-plugin/plugin.json`、`.cursor-plugin/plugin.json` 和 `.plugin/plugin.json`）更新为相同版本号。若其中任一文件不一致，`scripts/prepare_release.py` 将拒绝该标签。
2. 运行 `pytest tests/unit`。如有迁移说明，请一并写入 PR。
3. 合并后，维护者在选定的 `main` 提交上创建并推送带附注的 `vMAJOR.MINOR.PATCH` 标签。切勿移动已存在的发布标签。
4. 从 `main` 分支运行 **Prepare draft release** 工作流，并传入该已存在的标签。被打标签的提交必须包含发布脚本并通过单元测试。
5. 检查草稿中的两个运行时 ZIP 包、`SHA256SUMS`、来源证明包（`egm-vMAJOR.MINOR.PATCH.intoto.jsonl`）以及源码提交。按如下步骤验证两个 ZIP 包的证明（attestation）。将占位发布说明替换为变更列表、兼容性/迁移详情以及确切产物的宿主验证证据。未经验证的宿主仍保持未验证状态。
6. 仅在完成检查后手动发布该草稿。应用目录（Directory）的提交流程另行独立进行。

该工作流不会创建标签、覆盖已有发布、自动公开发布，也不会运行付费的实时测试。重复的发布尝试会直接失败，而不会替换现有发布。若构建失败，请先修复问题再创建新的候选版本。本地准备候选包时，请使用全新的输出目录：

```sh
python scripts/prepare_release.py --tag v0.1.0 --commit "$(git rev-parse HEAD)" --output /tmp/egm-0.1.0-candidate
```

本地命令会检查清单版本号并构建产物，但不会校验调用者提供的源码 SHA、Git 标签祖先关系或运行测试——这些检查属于 CI 工作流职责。仅在完成精确产物检查后，才可发布由工作流构建的候选版本。本地构建仅作为诊断用候选产物。精确的已安装包验证仍在 [#60](https://github.com/google/elephant-goldfish-model/issues/60) 中持续跟进。

## 验证下载产物

将两个 ZIP 包、`SHA256SUMS` 以及来源证明包下载到同一个目录中。在已安装 GitHub CLI 的环境下，于该目录运行以下命令：

```sh
sha256sum -c SHA256SUMS
for archive in egm-claude-*.zip egm-openai-*.zip; do
  gh attestation verify "$archive" --repo google/elephant-goldfish-model \
    --signer-workflow google/elephant-goldfish-model/.github/workflows/release.yml \
    --source-ref refs/heads/main || exit 1
done
```

工作流会在创建草稿之前对两个 ZIP 包的摘要进行签名，并将来源证明存储在 GitHub 上。同一份证明会作为 `egm-<tag>.intoto.jsonl` 附在发布页中，它不会列在 `SHA256SUMS` 中。如需使用下载的证明包验证某个 ZIP，请通过 `--bundle` 参数传入：

```sh
gh attestation verify egm-claude-0.1.2.zip --repo google/elephant-goldfish-model \
  --bundle egm-v0.1.2.intoto.jsonl
```

并按上文添加相同的 `--signer-workflow` 和 `--source-ref` 参数。

签名失败会阻断草稿创建。本地准备不会生成证明。构建作业使用 `id-token: write` 请求签名证书，并使用 `attestations: write` 存储结果；该作业没有写入发布（release）的权限。

来源证明标识的是从 `main` 分支触发的工作流运行记录。该次运行中检出的带标签源码可能是另一个提交。发布前请核对发布说明中的源码 SHA 并检查工作流运行记录。证明本身既不代表宿主支持，也不能证明代码绝对安全。
