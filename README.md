# Transfer2me：云端回传的收件箱

> 位置：`paper/p01-opd-teacher/Transfer2me/`（Windows 为 `D:\Code\CLAUDE\workspace\paper\p01-opd-teacher\Transfer2me\`）。

**传输方式（2026-11-06 你定的）**：
- 一次提交的实验，一次回传就带回全部所需的数据。
- 回传就是固定的 10 个文件，按数据类别分好，**直接放在本目录 `Transfer2me/` 下，不建回传子目录**。
- **所有传输都不限字数**：回传的文件和发给云端的传输包都一样，不切分，单个文件也不设上限，云端尽可能把数据写全。

本文件第 1 节是发给云端的完整要求，请把第 1 节发给云端 agent，或者直接把本文件传过去。第 2–4 节写的是你和我这边怎么收、怎么用。

---

## 1. 发给云端：回传要求

> 研究者 2026-11-06。本节替代 v1 包里所有关于回传的规定，其余照旧。被替代的有：
> - `docs/05` 第 3 节第 3 条；
> - `docs/06`，包括“每次最多 5000 字”“每段不超过 4500 字”的限制；
> - `docs/07`；
> - 各路线卡里的“回传文件”和“回传点”；
> - `make-return-text.py` 的分段打包。

### 1.1 总则

1. **只回传一次**：v1 的全部任务做完后，一次回传。我只根据这次回传来分析结果、安排下一版实验，不会再问你问题。所以凡是分析和下一步可能用到的，都要写进来。
2. **固定的 10 个文件**：把第 1.2 节的 10 个文件直接写在 `$OPD_RUNTIME/Transfer2me/` 下，文件名一字不改。研究者会把它们原样放进本机的 `Transfer2me/`。
   - 不建 `return-…` 之类的子目录，不另加文件。
   - 某个文件没有内容时，也要建出来：CSV 只写表头，md 写“无”。
   - `00-summary.md` 的前两行写：`回传编号：return-YYYYMMDD-NN`（日期是交付日，NN 从 01 起），以及 `包版本：<VERSION>`。
3. **不切分、不压缩、不限字数**：
   - 单个文件也不设上限；不用 `make-return-text.py`，不加分段标记。
   - 不降采样，不截断，数字保留测得的精度。
   - 每个配置都写，包括失败和 OOM 的；每次重复、每个种子、每一步、每条回答都写。
4. **格式**：
   - 编码都用 UTF-8；CSV 用逗号分隔，第一行是表头，列名和顺序按第 1.2 节。
   - 含逗号的字段加双引号；没有值的写 `NA`，并在 `note` 列或 `00-summary.md` 写原因。
5. **单位与命名**：
   - 单位：时间 s，显存和内存 GB，带宽 GB/s，吞吐 tok/s；比例写小数，不写百分号。
   - `task` 写 `state/tasks.yaml` 里的任务号；`cfg` 是配置名，在 10 个文件里保持一致。
6. **只写实测值**：估算要标“估算”。预热步照样写进去，标出来；统计均值时去掉预热。
7. **不写敏感信息**：不写凭据、令牌、账号。单位内部的代码只写改了什么，不贴原文；我们对开源仓库的改动要贴原文。

### 1.2 十个文件及格式

| # | 文件 | 类别 | 内容 |
|---|---|---|---|
| 0 | `00-summary.md` | 结论 | 任务表、失败与异常、归因、观察、学习要点、对下一步的建议、资源情况 |
| 1 | `01-env.yaml` | 环境与版本 | 硬件、拓扑、各环境的包版本、模型、数据、评测设置、版本漂移 |
| 2 | `02-code.md` | 代码与命令 | 每个 cfg 的完整命令、服务启动命令、环境变量、补丁原文、仿真参数 |
| 3 | `03-configs.csv` | 配置 | 每个配置一行 |
| 4 | `04-steps.csv` | 每步 | 每个训练步一行 |
| 5 | `05-metrics.csv` | 指标 | 不是脚本 JSON 的其他数值：标量和曲线 |
| 6 | `06-rollouts.csv` | 逐条回答 | 每步每条回答一行 |
| 7 | `07-timeline.csv` | 生成进度 | 每条回答的生成进度点 |
| 8 | `08-script-outputs.jsonl` | 脚本原始输出 | 包里脚本生成的每个 JSON，一行一个，原样 |
| 9 | `09-logs.md` | 日志 | 报错、OOM、服务启动、卸载、版本核对等日志原文 |

**`00-summary.md`**：前两行写回传编号和包版本（见第 1.1 节第 2 条），然后按下面的小节写：
1. `## 任务表`：`任务 | 状态(done/partial/blocked) | 开始–结束（北京时间） | 分区 | GPU·h | 结果（几句话，带关键数字） | 偏差与问题`。
2. `## 失败与异常`：每项写任务、cfg、现象、原因判断、试过的办法和结果；日志原文在 `09-logs.md`，这里写它的小节名。
3. `## 归因`：写给改了代码的任务（R1-T4、R2-2），以及所有性能对比。
   - 改了哪些文件和函数；
   - 关掉开关时与基线是否逐值一致，最大差是多少；
   - 对照是否只差开关，是否交替运行，各跑了几次；
   - 混杂因素逐项的结论：版本、缓存与预热、批大小与并发、调优、后台负载、频率与温度、计时开销、随机性；
   - 提升由哪些数据支撑；
   - AUDIT 的结论。
4. `## 观察`：瓶颈在哪里，和路线卡的“预期”有什么不同，有哪些意外现象。
5. `## 学习要点`：逐条回答各路线卡末尾的问题。
6. `## 下一步建议`：每条写做什么、依据哪些数据、预计 GPU·h。
7. `## 资源`：
   - 磁盘剩余空间；
   - 已下载的模型及路径；
   - 能用的环境（env_id 与路径）；
   - 代码仓库与 commit（`$OPD_RUNTIME/code/*`）；
   - 原始结果的目录。

**`01-env.yaml`**：
```yaml
hw:
  gpu: ""              # 型号、数量、每卡显存
  driver: ""
  cuda: ""
  nvlink: ""           # nvidia-smi nvlink -s 的摘要：链路数 × 速率
  topo: |              # nvidia-smi topo -m 原文
  pcie: ""             # 每卡的代数与宽度
  cpu: ""              # 型号、插槽数、核数
  numa: ""             # 节点数、每个节点的内存、各 GPU 亲和的 NUMA 节点
  host_mem_gb: 0
  os: ""
  kernel: ""
envs:                  # 每个环境一项
  <env_id>: {path: "", python: "", packages: {}}   # packages：pip freeze 里与实验有关的全部包；git 安装的写 commit
models:                # 每个模型一项
  - {name: "", repo: "", revision: "", local_path: "", size_gb: 0, dtype: "", smoke_test: ok}
data:
  - {name: "", version: "", split: "", n: 0, local_path: "", sampling: ""}
eval: {tool: "", version: "", benchmark: "", n_questions: 0, samples_per_q: 0, temperature: 0, top_p: 0, max_len: 0}
drift: []              # 版本漂移、换过的版本，以及原因
```

**`02-code.md`**：每个任务一个 `## <任务>` 小节。
- 每个 cfg 实际执行的完整命令，包括服务启动命令。可以从 `logs/*.cmd` 抄。
- 与实验有关的环境变量。
- 对开源仓库的改动：用 `git diff` 原文，例如计时补丁、R2-2 原型、对框架的其他修改。
- R2-P3 每次运行用的 `p3-params.json` 原文，每个参数注明取自哪次测量。

**`03-configs.csv`**：每个配置一行，包括失败的。

```
task,cfg,env,framework,student,teacher,teacher_prec,kv_dtype,placement,scoring_mode,tp,gpus,partition,max_len,group_size,prompts_per_step,steps,seeds,chunk,bg,chk,status,step_s,gen_s,tail_s,train_s,sync_s,peak_mem_gb,oom,args,note
```
- `bg`：测量时另一分区在做什么，写 `idle` 或任务号。
- `chk`：w = 已去预热，t = 基线调过参，n = 数值核对过，可以组合。
- `status`：ok / failed / oom。
- `step_s` 到 `sync_s`：去掉预热后的均值；`tail_s` 是生成结束到打分结束的时间。不测步时间的配置留空。
- `args`：全部非默认参数。

**`04-steps.csv`**：R1-T4、R1-E0、R1-E1、R2-2、R3-F1、R4-Q1 的每一步，包括预热步。

```
task,cfg,seed,step,warmup,step_start_unix,gen_start_s,gen_end_s,score_start_s,score_end_s,train_start_s,train_end_s,sync_start_s,sync_end_s,step_end_s,n_rollouts,n_prompts,tokens_generated,tokens_scored,len_p50,len_p90,len_p99,len_max,n_truncated,teacher_busy_s,peak_mem_gb_student,peak_mem_gb_teacher,loss,eval_acc,mode,note
```
- 时刻都从该步生成开始计时，和 `templates/log-schemas.md` 一致。
- `eval_acc` 只在评测的那一步填。
- `mode` 写打分方式：sync、complete、tail16、i8 等。

**`05-metrics.csv`**：不是脚本 JSON 的其他数值都放这里。

```
task,cfg,key,x_name,x,value,unit,n,lo,hi,src,note
```
- 标量的 `x_name`、`x` 留空；曲线每个点一行。
- `n` 是重复次数或种子数，`lo`、`hi` 是范围，只测一次时留空。
- `src` 写云端原始文件的路径。

**`06-rollouts.csv`**：R1-E0、R1-E1、R2-2、R4-Q1 每一步的每条回答。

```
task,cfg,seed,step,rollout_id,group_id,prompt_len,final_len,t_done_s,truncated,streamed,tokens_scored,note
```

**`07-timeline.csv`**：R1-E0 的全部步、R2-2 的全部步。用 `templates/log-schemas.md` 的 `rollout_timeline.csv` 原始粒度（每 64 个 token 一点，完成时一点）。

```
task,cfg,seed,step,rollout_id,group_id,prompt_len,n_tokens,t_s
```

**`08-script-outputs.jsonl`**：包里脚本写出的每个 JSON 文件都放进来，每个一行，内容原样，不改字段：

```
{"task": "R2-P1", "cfg": "c256", "file": "results/R2-P1/return/p1-chunk256.json", "data": {…脚本输出原样…}}
```

涉及的脚本：`p0-bandwidth-check.py`、`p1-incremental-scoring.py`、`p2-residency-sweep.py`（run 和 compare）、`p3-i8-offline-sim.py`（完整输出，包括 `per_step`）、`p4-fidelity-compare.py`、`p7-demotion-cost.py`，以及 R0-P5 的 `tokenizer-check.json`。

**`09-logs.md`**：每段一个 `## <任务> / <cfg> / <内容>` 小节，注明原始日志的路径，正文放在代码块里，原样。至少包括：
- 每个报错的完整 traceback；
- OOM 前后的日志；
- 每次教师服务启动时关于显存、KV 池、CUDA graph、卸载的日志（卸载了多少 GB、哪些层）；
- 每个任务开始前 `check-versions.py` 或 `env-snapshot.sh` 的输出；
- 框架启动时打印的完整配置。

### 1.3 每个任务要的数据

下面是**至少**要有的，多写不限。文件号对应第 1.2 节。

**通用**：
- 每个配置都在 `03` 里写一行，在 `02` 里写它的命令。
- 每次启动教师服务，都在 `05` 里写 `kv_pool_tok`、`kv_bytes_per_tok`、`max_total_tokens`、`mem_fraction_static`；cfg 写服务名，数值从启动日志读。
  - GLM 另写每条回答的固定状态字节数 `state_bytes_per_seq` 和 indexer 键每 token 的字节数 `idx_bytes_per_tok`。
  - 启动日志原文放进 `09`。

**R0-P0 硬件**：
- `01` 的 `hw` 全部字段；
- 两个带宽 JSON 放进 `08`；
- 跑过 nvbandwidth 的话，结果写进 `05`（key 写 `nvbw_<测试名>`）；
- `00` 里把单向 P2P 和此前 E15 的 339.8 GB/s 对照，说明差异的原因。

**R0-T1 环境与示例**：
- `01` 的 `envs`；
- verl 和 slime 的两个 OPD 示例，在 `03` 里各一行，在 `04` 里写每一步，在 `05` 里写 `steps_run`、`ok`；
- `00` 里写两个框架中教师 logprob 进入损失的位置（文件:函数）和格式，以及教师放在哪里。

**R0-T2 模型与数据**：
- `01` 的 `models`、`data`；
- 每个模型的打分冒烟测试，结果写在 `smoke_test` 字段。

**R0-P5 tokenizer**：
- 原始 JSON 放进 `08`；
- `05` 里写 `files_identical:<文件名>`（0/1）、`encode_mismatch_count`、`decode_mismatch_count`、`n_texts`；
- 有不一致时，把前 20 个例子放进 `09`。

**R2-P1 增量打分**：
- 4 个 JSON（c256、c1024、det-c256、det-c1024）放进 `08`，包括 `per_sequence`；
- 两次服务启动的命令写进 `02`；
- `00` 里写三个判断：前缀是否复用，偏差的量级，增量打分比一次性打分多花多少时间。

**R1-T4 计时改造**：
- 补丁原文放进 `02`；
- 开、关计时各 5 步，在 `03` 里各一行，在 `04` 里写每一步；
- `05` 里写 `timing_overhead`（开计时的步时间 / 关计时的步时间 − 1）。

**R1-E0-S、R1-E0-L 基线 OPD**：
- `03`：每个运行一行，写明 L 负载选了哪种放法（学生 LoRA 或教师 FP8）；如果跑了 verl 对照，单列一个 cfg。
- `04`：全部步。`06`：全部回答。`07`：全部时间线。
- `05` 里写：
  - `trunc_frac`；
  - `teacher_idle_gen_s`：生成阶段教师的空闲时间，每步一个点，x 写 step；
  - `t_done50_s`、`t_done90_s`、`t_done100_s`：每步一个点；
  - 生成阶段 GPU 利用率随时间的变化：`gpu_util`，x 写 t_s，能采就采；
  - 第 0 步和最后一步的 `math500_acc`，以及 `n_questions`。

**R1-E1 区间网格**：
- 约 20 个配置，每个在 `03` 里一行，包括 OOM 和跑不了的；
- `04`：每个配置的全部步；`06`：全部回答；
- `05` 里写每个配置的 `tok_s_student`、`tok_s_teacher`、`teacher_idle_gen_s`；
- `00` 里列出网格的中心配置和每个维度的取值，以及没跑的配置及原因。

**R2-P7 降级代价**：
- hicache 和 nohicache 两个 JSON 放进 `08`；
- `--flood-tokens` 的取值和依据写进 `02`；
- `05` 里写 `L_star`：恢复比重算快的最短长度。

**R3-P2 稠密教师持平点**：
- 5 个设置（off0、off8、off16、pf4-2、pf4-3）的 run JSON 和 2 个 compare JSON 放进 `08`；
- 每个设置在 `03` 里一行，启动失败的也写；
- `05` 里写每个设置的 `host_mem_gb`、`gpu_mem_gb`、`offloaded_gb`、`offloaded_layers`；
- 卸载日志放进 `09`。

**R3-P2-MoE**：
- Qwen3-30B-A3B 和 GLM-5.3-Flash 各个设置的写法同 R3-P2，cfg 写 `<模型>-<设置>`；
- `00` 里写 GLM 用的是 SGLang 还是 vLLM、哪种卸载方式，以及不能用的原因。

**R3-F1 前沿对**：
- `03` 一行，`04` 写每一步；
- `05` 里写每张 GPU 的显存（`gpu_mem_gb`，x 写 GPU 号，分学生和教师）、`host_mem_gb`、`ok_steps`；
- `00` 里写显存估算与实测的对照；跑不通的话，写清卡在哪一步、缺什么。

**R2-P3 离线仿真**：
- 每次运行（S、L × chunk 256、1024 × KV 预算原值、加倍）的完整 JSON 放进 `08`，包括 `per_step`；
- 参数文件放进 `02`；
- 合成数据的自测结果写进 `05`。

**R4-P4 逐 token 偏差**：
- 每个对比的 JSON 放进 `08`；做了 32B 的也一起放；
- 参照服务和被测服务的启动参数写进 `02`；
- 固定序列的条数和长度写进 `05`。

**R4-Q1 短程训练**：
- BF16 和 FP8 各 2 个种子，每个在 `03` 里一行；
- `04` 写全部步，包括 loss、每 25 步的 `eval_acc`；
- `06` 写全部回答。

**R2-2 选择性流式原型**（R2-P1 显示前缀复用生效，且 R2-P3 的 `i8_gain_vs_best_baseline` ≥ 0.10 时做）：
- 设计说明写进 `00`：选择规则、Δ、预算、安全系数、降级方式；
- 补丁原文放进 `02`；
- 每个负载、长度、模式（base、tail16、i8、i8-off，i8-off 即开关关闭）在 `03` 里一行；`04`、`06`、`07` 写全；
- `05` 里写：
  - `peak_teacher_kv_gb`、`n_demote`、`n_streamed`、`tokens_recomputed`；
  - `logprob_max_abs_diff`、`loss_max_abs_diff`（20 步，与基线比）；
  - 每次决策的耗时 `decide_ms`，写 p50、p99、max。

**R5-P8、R5-P9**（可选）：
- P8 的每个教师、每种方法、每次重复，在 `05` 里写 `swap_s`，cfg 写 `<教师>-<方法>`；
- 换入后第一次打分的延迟写 `first_score_s`；
- P9 的计算过程和结果写进 `00`，数字写进 `05`。

### 1.4 交付前自检

1. `$OPD_RUNTIME/Transfer2me/` 下正好是第 1.2 节的 10 个文件，文件名一字不差，没有子目录；`00-summary.md` 的前两行写了回传编号和包版本。
2. `state/tasks.yaml` 的每个任务在任务表里都有一行。
3. 第 1.3 节每一项都有数据，或者写了 NA 和原因。
4. `03` 的每个 cfg，在其他文件里都能找到它的数据；其他文件里出现的 cfg，在 `03` 里都有一行。
5. CSV 每行的列数与表头一致；`08` 每行都是合法的 JSON。
6. 交付后告诉研究者回传编号，以及每个文件的大小。

---

## 2. 收件（你）

把云端 `$OPD_RUNTIME/Transfer2me/` 下的 10 个文件原样复制到本目录（`paper/p01-opd-teacher/Transfer2me/`），和本 README 放在一起。不改名，不改内容。
- 上一次回传的文件，我在处理时已经存档到 `../data/<回传编号>/`，所以新文件直接覆盖即可。

## 3. 本机处理（我）

1. **核对**：运行 `python3 -I paper/skills/tools/transfer2me-check.py paper/p01-opd-teacher/Transfer2me --tasks paper/p01-opd-teacher/Transfer2byd/v1-20261104/state/tasks.yaml`。它检查：
   - 10 个文件是否齐全，有没有多余的文件（README 除外）；
   - 各 CSV 的表头与列数；
   - JSONL 能否解析；
   - cfg 是否对得上；
   - 是否覆盖了全部任务；
   - 回传编号。

   输入只当数据读取，不执行。
2. **存档**：按 `00-summary.md` 里的回传编号，把 10 个文件原样复制到 `../data/<回传编号>/`，只追加，不改。之后的分析都读存档。
3. **分析**：按 research-router 第 9 节对照 `guidance/02`、假设 H1–H6 和判断节点。
   - 成本常数（t₀、s、s_a、T₀、恢复带宽）我用 `08` 的原始点拟合；
   - R1-E0 的负载我用 `06`、`07` 在本机回放，新的仿真策略也在本机先试。
4. **安排下一版**：按 `paper/skills/07-云端传输包编写规范-2026-11-05.md` 出下一版传输包。以后每一版都沿用这 10 个文件和这种放法，在包的 `docs/06–07` 里写好这一版每个任务要的数据，写法同第 1.3 节。

## 4. 回传记录

| 回传编号 | 收到日期 | 包含的任务 | 处理结果 |
|---|---|---|---|
| （还没有） | | | |
