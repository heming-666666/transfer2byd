# Transfer2me：云端回传的收件箱

> 位置：`paper/p01-opd-teacher/Transfer2me/`（Windows 为 `D:\Code\CLAUDE\workspace\paper\p01-opd-teacher\Transfer2me\`）。v1 包结束时提示的 `paper/Transfer2me/` 已过时，放到这里。

## 1. 回传要求（2026-11-06 起，v1 也适用）

你的要求：
- 一次回传就把全部结果、配置和版本传回来；
- 不限字数，让云端尽可能把数据写全；
- 文件按规则分类，总数不超过 10 个。

下面“转发开始”到“转发结束”之间的内容，请整段发给云端 agent。一条消息放不下时，分成两条发。它替代 v1 包里的 `docs/05` 第 3 节第 3 条（每 1–2 天回传一次）、`docs/06` 第 2、4、5 节和 `docs/07`。

**——转发开始——**

**回传要求（研究者 2026-11-06；替代 docs/05 第 3 节第 3 条、docs/06 第 2、4、5 节和 docs/07，其余照旧）**

**一、一次交齐**

v1 的全部任务做完后，一次回传，不分批。我收到后直接分析，并安排下一版实验。
- **写全**：不限字数，单个文件也不设上限，尽可能写全。
  - 每个配置都写，包括失败的；每次重复、每个种子、每一步都写。
  - 曲线不降采样；数字保留测得的精度。
- **分段**：`make-return-text.py` 会把内容切成每段不超过 4500 字，这只是为了便于复制传输，不限制内容。

**二、文件：按类分，共 9 个**

9 个文件都放在 `$OPD_RUNTIME/results/RETURN/return/`。每个数据行都带 `task` 和 `cfg` 两列，`cfg`（配置名）在所有文件里保持一致。

| 文件 | 类别 | 放什么 |
|---|---|---|
| `00-summary.md` | 结论 | 见下 |
| `01-env.yaml` | 环境 | 硬件、拓扑、各环境的完整版本、模型、数据、评测设置、版本漂移 |
| `02-code.md` | 代码 | 命令、启动参数、补丁 |
| `03-configs.csv` | 配置 | 每个配置一行 |
| `04-steps.csv` | 每步 | 每个训练步一行 |
| `05-scalars.csv` | 标量 | 单个数值的结果 |
| `06-curves.csv` | 曲线与分布 | 随某个量变化的结果 |
| `07-items.csv` | 逐条 | 逐条回答、逐条序列、逐次重复的原始记录 |
| `08-logs.md` | 日志 | 报错、警告和关键日志的原文 |

**打包**：运行 `python3 scripts/make-return-text.py --tasks RETURN --max-lines 1000000 --note "<一句话>"`。`--max-lines` 的作用是不让脚本截断长表。

**通用规则**：
- 只写实测值；没测的写 NA，并写明原因。
- 单位：s、GB、GB/s、tok/s；比例写小数。
- 算均值时去掉预热步；预热步本身也要写进 `04-steps.csv`，标 `warmup=1`。
- 同一组对比只用一个环境。

**三、各文件的写法**

1. **`00-summary.md`**
   - **任务表**：`任务|状态|开始–结束|GPU·h|结果|偏差与问题`。
   - **失败与异常**：每项写现象和原因；日志原文放 `08-logs.md`。
   - **归因**（R1-T4、R2-2）：
     - 改动的文件和函数；
     - 关掉开关时与基线是否逐值一致，最大差是多少；
     - 对照是否只差开关，是否交替运行；
     - 混杂因素逐项的结论；
     - 提升由哪些数据支撑；
     - AUDIT 的结论。
   - **观察与建议**：瓶颈在哪里，和预期有什么不同；对下一步实验的建议，每条附依据和预计 GPU·h。
   - **资源**：磁盘剩余空间、已下载的模型及路径、能用的环境，供安排下一版用。
2. **`01-env.yaml`**
   - `hw`：GPU、驱动、CUDA、NVLink、`nvidia-smi topo -m` 原文、PCIe、CPU、NUMA、内存、OS、内核；
   - `envs`：每个环境写 env_id，以及 pip freeze 里与实验有关的全部包和版本或 commit；
   - `models`：name、repo、revision、大小、dtype、路径；
   - `data`：名称、版本、划分、条数、采样方式；
   - `eval`：工具、版本、每题采样次数、temperature、top_p、最大长度、题数；
   - `drift`：版本漂移和换版本的记录，以及原因。
3. **`02-code.md`**
   - 每个 cfg 实际执行的完整命令；
   - 教师服务的完整启动命令；
   - 环境变量里与实验有关的项（不含凭据）；
   - 对开源仓库的全部改动，给 `git diff` 原文（计时补丁、R2-2 原型）；
   - 单位内部的代码只写改了什么。
4. **`03-configs.csv`**
   `task,cfg,env,student,teacher,prec,kv,place,mode,tp,gpus,partition,max_len,group,prompts,steps,seeds,bg,chk,status,args,step_s,gen_s,tail_s,train_s,sync_s,mem_gb,oom,note`
   - `bg`：测量时另一分区在做什么，写 `idle` 或任务号。
   - `chk`：w = 已去预热，t = 基线调过参，n = 数值核对过。
   - `args`：全部非默认参数。
   - 步时间各列给测步时间的配置填均值；`tail_s` 是生成结束到打分结束的时间。
5. **`04-steps.csv`**：R1-E0、R1-E1、R2-2、R4-Q1 的每一步，包括预热步。
   `task,cfg,seed,step,warmup,gen_s,score0_s,score1_s,train_s,sync_s,step_s,tok_gen,tok_scored,n_rollouts,len_p50,len_p90,len_p99,len_max,n_trunc,busy_s,mem_s,mem_t,loss,acc`
   - 时刻都从该步生成开始计时；`score0_s`/`score1_s` 是第一次打分请求开始、最后一次完成的时刻。
6. **`05-scalars.csv`**：`task,cfg,key,value,unit,n,lo,hi,src`
   - `n` 是重复次数，`lo`、`hi` 是范围；
   - `src` 写云端原始文件的路径。
7. **`06-curves.csv`**：`task,cfg,key,x_name,x,value,n,lo,hi`，每个点一行，不降采样。
8. **`07-items.csv`**：`task,cfg,item_type,step,id,group,a,b,c,d,note`。
   - `item_type` 决定 a–d 四列的含义，在 `00-summary.md` 里列一张对照表。
   - 至少包括下面三类：
     - `rollout`（R1-E0、R2-2，每步每条回答）：`a`=提示长，`b`=最终长度，`c`=完成时刻 s，`d`=是否截断；
     - `seq`（R2-P1，每条序列）：`a`=提示长，`b`=回答长，`c`=一次性打分 s，`d`=增量打分 s；
     - `rep`（R2-P7、R3-P2 的每次重复）：`a`=L 或 pass_tokens，`b`=耗时 s，`c`=吞吐，`d`=缓存命中 token。
9. **`08-logs.md`**：每段日志前注明任务、cfg 和原始文件路径。内容包括：
   - 报错的完整 traceback；
   - OOM 前后的日志；
   - 教师服务启动日志里的 KV 池、显存分配；
   - 版本核对的输出。

**四、各任务至少要有的 key**（写在 05/06 里，多写不限）

- 每次启动教师服务（cfg 写模型名）：`kv_pool_tok`、`kv_bytes_tok`、`state_bytes_seq`、`idx_bytes_tok`（GLM 的 indexer 键）。
- R0-P0：`h2d`、`d2h`（8 卡并发时的每卡值）、`h2d_agg`；`p2p` 分方向，x 写 GPU 对。
- R0-P5：`tok_same`、`enc_mis`、`dec_mis`、`n_texts`。
- R2-P1（cfg 为 c256、c1024、det-c256、det-c1024）：`diff_p50`、`diff_p99`、`diff_max`、`bad_seqs`，逐条序列放 07。
- R1-T4：`timing_overhead`。
- R1-E0-S/L：
  - `len_p50/p90/p99/max`、`trunc_frac`、`t50_s`、`t90_s`、`t100_s`、`idle_gen_s`、`n_q`；
  - 曲线：`done_frac` 随 t 变化（每一步），`acc` 随 step 变化。
- R1-E1：`tok_s_stu`、`tok_s_tea`。
- R2-P7：`cold_s`、`hit_s`、`evict_s` 随 L 变化。
- R3-P2、R3-P2-MoE：`tok_s`、`max_req_s` 随 pass_tokens 变化；每个 compare 一个 `t_star`；`host_mem_gb`。
- R2-P3（cfg 为 S、L、S-kv2、L-kv2）：
  - `end_s`、`tail_s`、`feasible` 按策略分（x 写策略名）；
  - `gain_vs_sync`、`gain_vs_complete`、`gain_vs_best`、`oracle_gain_vs_best`；
  - 参数及其来源。
- R4-P4（cfg 写对比名）：`diff_p50/p90/p99/max`、`top1`、`kl`。
- R3-F1：`ok_steps`、`host_mem_gb`，以及每张 GPU 的 `mem_gb`。
- R2-2（模式为 base、tail16、i8、i8-off）：`t_kv_gb`、`n_demote`、`lp_maxdiff`、`loss_maxdiff`、`decide_ms`。
- R4-Q1：`loss`、`acc` 随 step 变化（cfg 写 `精度-s种子`）。
- R5-P8：`swap_s`，按教师和方法分。

**五、打包前自检**

1. `state/tasks.yaml` 的每个任务在任务表里都有一行。
2. 第四部分的每个 key 都有值，或写了 NA 和原因。
3. `03-configs.csv` 的每个 cfg，在其他文件里都能找到它的数据。
4. 每个计时的 cfg 都写了 `bg`、`chk`。

打包后告诉研究者回传编号和段数。

**——转发结束——**

**为什么这样定**：
- **按数据形状分 9 类**：结论、环境、代码、配置、每步、标量、曲线、逐条、日志。列是固定的，本机可以直接读入。
- **全量回传**：本机可以拿原始点自己拟合成本常数（t₀、s、s_a、T₀、恢复带宽），用逐条回答记录回放 R1-E0 的负载，用每次重复的数据判断波动。
- **能对上**：`guidance/02` 第 1、2 节的每个数，都对应第四部分的一个 key 或一类逐条记录。

## 1.1 以后每一版都照这个原则写

- **默认云端会做完**：默认云端会把包里的实验基本都做完，一批实验回传一次。
- **回传格式**：不限字数；文件沿用上面 9 类，总数不超过 10 个。
- **`docs/07` 列 key**：在包的 `docs/07` 列出每个任务“至少要有的 key”，每个 key 都要对得上 `guidance/02` 的一行或一个判断节点。
- **回传后**：按 research-router 第 9 节分析，然后安排下一版实验。

## 2. 放法

每次回传建一个子目录，目录名用回传编号（每段第一行 `<<<BEGIN RETURN return-YYYYMMDD-NN PART 01/NN>>>` 里的编号）：

```
Transfer2me/return-20261106-01/
  part-01.md … part-NN.md   ← 每段一个文件；也可以按顺序全部粘进一个 all.md
  notes.md                  ← 可选：云端 agent 在对话里说的情况
```

- 每段从 `<<<BEGIN …>>>` 到 `<<<END … sha12=…>>>` 原样保留。行尾空格和换行符的差异不影响校验。
- 缺段、重复或被截断时，拼装脚本会指出是哪一段。请云端重发那一段即可，分段还在 `$OPD_RUNTIME/returns/<回传编号>/`。
- 不要放凭据、单位内部资料或与实验无关的文件。

## 3. 本机怎么处理

1. 运行 `python3 -I paper/skills/tools/transfer2me-assemble.py paper/p01-opd-teacher/Transfer2me/<回传编号>`，核对校验码。9 个文件会还原到 `tmp/transfer2me/<回传编号>/files/RETURN/return/`。
2. 按 research-router 第 9 节入库到 `data/<回传编号>/`，对照 `guidance/02`，再更新研究文档和论文。只用实测数字。
3. 按 `paper/skills/07-云端传输包编写规范-2026-11-05.md` 出下一版传输包。

## 4. 回传记录

| 回传编号 | 收到日期 | 包含的任务 | 处理结果 |
|---|---|---|---|
| （还没有） | | | |
