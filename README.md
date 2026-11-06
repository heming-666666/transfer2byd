# Transfer2me：云端回传的收件箱

> 位置：`paper/p01-opd-teacher/Transfer2me/`（Windows 为 `D:\Code\CLAUDE\workspace\paper\p01-opd-teacher\Transfer2me\`）。v1 包结束时提示的 `paper/Transfer2me/` 已过时，放到这里。

## 1. 回传要求（2026-11-06 起，v1 也适用）

你的要求：几个文件就能回传全部所需的数据和配置，字数尽量少，保留关键数据和表述。从“转发开始”到“转发结束”整段发给云端 agent，一条消息就能发完（不到 4500 字）。它替代 v1 包里 `docs/06` 第 2、4、5 节和 `docs/07`。云端的任务记录（`return.yaml`、原始日志）照旧写，但留在云端。

**——转发开始——**

**回传要求（研究者 2026-11-06；替代 docs/06 第 2、4、5 节和 docs/07，其余照旧）**

只回传 5 个文件，放在 `$OPD_RUNTIME/results/RETURN/return/`，然后运行 `python3 scripts/make-return-text.py --tasks RETURN --max-lines 5000 --note "<一句话>"`（`--max-lines` 防止长表被截断）。各任务的 `return.yaml`、`return/`、原始日志和完整命令留在 `results/<任务>/`，不回传，我需要时再问。

通用规则：
- 只写实测值；没测的写 `NA`，并在 summary 里写原因。
- 单位：s、GB、GB/s、tok/s；比例写小数；小数保留 3 位有效数字，整数原样写。
- 分布给 p50/p90/p99/max；曲线每条最多 50 点，均匀取点，保留首尾。
- 去掉预热；同一组对比只用一个环境。
- 配置名 `cfg` 在 5 个文件里保持一致；失败的配置也要写。

1. `summary.md`
   - 任务表：`任务 | 状态(done/partial/blocked/todo) | 一句话结果 | 偏差与问题`。OOM、报错、改了命令或计划、离预期很远的数字都要写。
   - `## 归因`：只写改了代码的任务（R1-T4、R2-2）：
     - 改了哪些文件和函数，增删多少行；
     - 关掉开关时与基线是否逐值一致，最大差是多少；
     - 对照是否只差开关，是否交替运行；
     - 提升由 data.csv 里哪些 key 支撑；
     - AUDIT 的结论。
   - `## 下一步`：最多 3 条，每条附理由。
2. `env.yaml`
   ```yaml
   hw: {gpu, driver, cuda, nvlink, cpu, numa, host_mem_gb}
   envs: {<env>: {python, torch, sglang, vllm, verl, slime, megatron, evalscope}}  # 版本或 commit
   models: [{name, repo, revision, size_gb}]
   data: [{name, version, n}]
   eval: {tool, version, samples_per_q, temperature, max_len}
   code: [{repo, base, head, diff_stat, by_agent}]  # 本地改动；不贴单位内部代码
   ```
3. `configs.csv`：每个配置一行。
   `cfg,task,env,student,teacher,prec,kv,place,mode,tp,gpus,max_len,group,prompts,steps,seeds,bg,chk,args,step_s,gen_s,tail_s,train_s,sync_s,mem_gb,oom`
   - `bg`：测量时另一分区在做什么，写 `idle` 或任务号。
   - `chk`：w = 已去预热，t = 基线调过参，n = 数值核对过。
   - `args`：关键启动参数，用 `;` 分隔。
   - 最后 7 列只给测步时间的配置填均值，其他配置留空。`tail_s` 是生成结束到打分结束的时间。
4. `steps.csv`：R1-E0 和以后的端到端实验，每步一行，每个 cfg 最多 50 行。
   `cfg,step,gen_s,score0_s,score1_s,train_s,sync_s,step_s,tok_gen,tok_scored,len_p50,len_p90,len_max,n_trunc,busy_s,mem_s,mem_t`
   - `score0_s`/`score1_s`：第一次打分请求开始、最后一次完成的时刻，都从该步生成开始计时。
   - `busy_s`：教师忙碌的时间。
5. `data.csv`：其余数字都放这里。列为 `task,cfg,key,x,value`，标量的 x 留空。

各任务的 key：
- 每次启动教师服务（cfg 写模型名）：`kv_pool_tok`、`kv_bytes_tok`、`state_bytes_seq`、`idx_bytes_tok`（GLM 的 indexer 键；日志里有才填）。
- R0-P0：`h2d`、`d2h`（8 卡并发时的每卡值）、`h2d_agg`；`p2p`（单向），x 写 `0-1`、`0-4`。
- R0-T1：每个示例一个 cfg，只填 configs。
- R0-P5：`tok_same`（0/1）、`enc_mis`、`dec_mis`、`n_texts`。
- R2-P1（cfg 为 c256、c1024、det-c256、det-c1024）：
  - `diff_p50`、`diff_p99`、`diff_max`、`bad_seqs`；
  - 每条序列一组，x 写序号：`P`、`R`（提示和回答的 token 数）、`one_s`、`inc_s`。
- R1-T4：`timing_overhead` = 开计时的步时间 / 关计时的步时间 − 1。
- R1-E0-S、R1-E0-L：
  - `len_p50`、`len_p90`、`len_p99`、`len_max`、`trunc_frac`、`n_q`；
  - `t50_s`、`t90_s`、`t100_s`：50%/90%/100% 回答完成的时刻，取各步均值；
  - `idle_gen_s`：生成阶段教师的空闲时间；
  - `done_frac`，x 写 t_s：取一个代表步的完成比例曲线，在 summary 里注明是哪一步；
  - `acc`，x 写 step。
- R1-E1：configs 里每个配置一行；data 里加 `tok_s_stu`、`tok_s_tea`。
- R2-P7：`cold_s`、`hit_s`、`evict_s`，x 写 L；多次重复的取中位数。
- R3-P2、R3-P2-MoE（cfg 写 `<模型>-<设置>`，如 qwen32b-off16）：
  - `tok_s`，x 写 pass_tokens；
  - `t_star`，每个 compare 一个；
  - `host_mem_gb`。
- R2-P3（cfg 为 S、L、S-kv2、L-kv2）：
  - `end_s`、`tail_s`、`feasible`，x 写策略名（sync、complete、stream_all、tail_k8…tail_k64、i8、oracle）；
  - `gain_vs_sync`、`gain_vs_complete`、`gain_vs_best`、`oracle_gain_vs_best`；
  - 参数及其来源写在 configs 的 args 里。
- R4-P4（cfg 写对比名，如 tp1-vs-tp2）：`diff_p50`、`diff_p90`、`diff_p99`、`diff_max`、`top1`、`kl`。
- R3-F1：
  - configs 里一行，填各阶段时间；
  - `ok_steps`、`host_mem_gb`；
  - `mem_gb`，x 写 GPU 号。
- R2-2（cfg 写 `负载-长度-模式`，模式为 base、tail16、i8、i8-off，i8-off 即开关关闭）：configs 里一行；data 里加 `t_kv_gb`、`n_demote`、`lp_maxdiff`、`loss_maxdiff`（20 步损失曲线的最大差）、`decide_ms`。
- R4-Q1（cfg 写 `精度-s种子`）：`loss`、`acc`，x 写 step。
- R5-P8（可选）：`swap_s`，cfg 写教师，x 写方法。R5-P9 的结论写进 summary。

一次回传一般不超过 12 段；超过 20 段说明曲线点太多，先降采样。G1 前先交 R1-E0-S/L、R2-P1、R2-P3、R2-P7、R3-P2。打包后告诉研究者回传编号和段数。

**——转发结束——**

**为什么这样定**：
- **能追溯**：5 个文件覆盖 `guidance/02` 第 1、2 节的每个数，以及 G1–G3 要用的数。原始文件留在云端 `results/<任务>/`，所以每个数仍能追到原件。
- **能自己拟合**：成本模型的常数由本机拟合，云端不用算。t₀、s、s_a 用 R2-P1 每条序列的数据，T₀ 和恢复带宽用 R2-P7 随 L 的变化。
- **不再回传**：路线卡的“学习要点”和各任务的完整 `return.yaml`，都不再传。

**以后的版本**：新增任务时，在包的 `docs/07` 里给它列 data.csv 的 key。每个 key 都要能对上 `guidance/02` 的一行或一个判断节点，对不上的就不回传。

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

1. 运行 `python3 -I paper/skills/tools/transfer2me-assemble.py paper/p01-opd-teacher/Transfer2me/<回传编号>`，核对校验码。5 个文件会还原到 `tmp/transfer2me/<回传编号>/files/RETURN/return/`。
2. 按 research-router 第 9 节入库到 `data/<回传编号>/`，对照 `guidance/02`，再更新研究文档和论文。只用实测数字。
3. 按 `paper/skills/07-云端传输包编写规范-2026-11-05.md` 出下一版传输包。

## 4. 回传记录

| 回传编号 | 收到日期 | 包含的任务 | 处理结果 |
|---|---|---|---|
| （还没有） | | | |
