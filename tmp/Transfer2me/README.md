# Transfer2me：云端回传的收件箱

云端 agent 按 `Transfer2byd/docs/06`、`07` 把结果整理好，再用 `Transfer2byd/scripts/make-return-text.py` 打包成若干段纯文字（每段不超过 4500 字）。你把这些文字原样带回这里；我拼装、核对、分析后，修改研究文档和论文，再更新 `Transfer2byd` 出新版本。

## 放法

每次回传建一个子目录，目录名用回传编号，即每段第一行 `<<<BEGIN RETURN return-YYYYMMDD-NN PART 01/NN>>>` 里的 `return-YYYYMMDD-NN`：

```
Transfer2me/
  return-20261106-01/
    part-01.md        ← 每段一个文件（文件名随意，扩展名 .md 或 .txt）
    part-02.md
    ...
    notes.md          ← 可选：你的补充说明，例如云端 agent 在对话里说的情况
```

- 也可以把所有段按顺序粘进同一个文件（例如 `all.md`），拼装脚本会自己切开；
- 每段从 `<<<BEGIN …>>>` 一行到 `<<<END … sha12=…>>>` 一行都要包括，中间不要改动；行尾空格、换行符的差异不影响校验；
- 少了段、重复粘贴、复制时被截断，拼装脚本都会指出是哪一段，让云端重发那一段即可（分段还在云端 `$OPD_RUNTIME/returns/<回传编号>/`）；
- 云端 agent 在对话里给的说明、截图里的数字，可以写进 `notes.md`，我会一起看；
- 不要放凭据、单位内部资料或与实验无关的文件。

## 我这边怎么处理

1. 运行 `python3 -I paper/shared/tools/transfer2me-assemble.py paper/Transfer2me/<回传编号>`：核对每段的校验码，把文件还原到 `tmp/transfer2me/<回传编号>/files/`；
2. 读各任务的 `return.yaml` 和结果文件，更新研究文档（`paper/p01-opd-teacher/research/`）和论文（`paper/p01-opd-teacher/manuscript/icml2027/`），只用实测数字；
3. 按结果修改 `Transfer2byd`（路线卡、任务清单、`CHANGELOG.md`），版本号加一，运行 `python3 -I paper/shared/tools/transfer2byd-build.py` 重建 `paper/Transfer2byd.zip`，交给你上传。

## 回传记录

| 回传编号 | 收到日期 | 包含的任务 | 处理结果 |
|---|---|---|---|
| （还没有） | | | |
