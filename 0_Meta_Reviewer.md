## Meta Reviewer

**Response:** 感谢 Meta Reviewer 对 AVE-Compass 的 benchmark 定位、多维解耦评测以及 AVE-Agent 规划与迭代编辑框架的认可。结合所有审稿人的意见，我们围绕统计稳健性、评测可靠性、Agent 组件贡献、实际开销、方法覆盖和复现性进行了系统补充。下面按新增证据组织，每组实验仅汇报一次。

### 1. Benchmark 规模、类别稳定性与聚合结果来源

AVE-Compass 包含来自 **85 个源视频的 190 条编辑指令**。33 个细粒度类别用于覆盖和失败定位，类别级模型比较则聚合到 13 个语义边界明确的父类别。我们在固定模型输出、输入样本和评测配置的条件下完成五次独立评测：主观指标在 0–100 量程上的最大极差为 **4.36**，所有 ICC 均不低于 **0.91**，客观指标的五次结果完全一致。样本较多的 J1、J4 和 J5 的整体分数极差分别仅为 **1.37、1.03 和 1.24**，类别排序在五次评测中保持一致。

我们还将聚合差值分解为按样本数加权的类别贡献。AVE-Agent 相比 Wan2.7 在 13 个父类别中的 **10 个**领先，相比 LTX2 在**全部 13 个**类别中领先；相对 Wan2.7 的主要收益来自 J1、J4 和 J6 等强跨模态任务，三类合计贡献 **6.36 分**。因此，总体提升同时具有评测运行稳定性和广泛的类别来源。

### 2. Checklist-guided MLLM 评测的分维度与分类别验证

我们将现有人工验证集中的 **1,173 个 checklist 判断**按评测维度和顶层任务类别拆分，并补充 Agreement、Balanced Accuracy、F1、Cohen’s κ 和 95% 置信区间。IF、Fidelity 和 Realism/AVQ 的一致率分别为 **96.42%、93.43% 和 96.74%**，对应 κ 为 **0.920、0.869 和 0.934**；总体一致率为 **95.48%**，κ 为 **0.908**。Joint、video-only、audio-only 和 speech 的类别级一致率均不低于 **95%**。这一分解表明，总体一致性同时保持在不同评分目标和不同编辑范围上，并非由某一高频维度或任务类别主导。

### 3. AVE-Agent 的受控消融与组件贡献

我们在同一评测子集上进行受控消融，所有变体使用**相同 Wan backbone、相同外部工具集合和相同输入**，仅改变规划、反馈利用、prompt refinement 和 retry-selection 策略。Full AVE-Agent 的总体得分为 **69.6**，Naive Planner、Blind Retry、No Prompt Refinement 和 No Evaluation/Retry 分别为 **45.1、65.8、64.9 和 64.0**。Full 相比 Naive Planner 的 **24.5 分**差异定位了结构化规划的作用；Full 与 Blind Retry 的差异表明定向使用 Evaluator 反馈比盲目增加候选更有效；No Prompt Refinement 则隔离了工具适配 prompt 的贡献。因此，性能收益来自对相同 backbone 和工具的结构化编排，而非模型容量或工具数量差异。

### 4. 延迟、工具调用、重试与 API 成本

在固定 Kling O3 Pro 后端的受控比较中，Direct Kling 和 AVE-Agent 的平均延迟分别约为 **7.7 和 12.0 分钟**，每条成本分别为 **US$1.68 和 US$1.90**。AVE-Agent 平均调用 3.4 次外部编辑工具和 3.6 次 Evaluator，53.6% 的任务触发 refinement，平均 retry 次数为 1.36。因此，额外计算主要按需分配给被 Evaluator 定位为存在执行或跨模态一致性问题的任务，API 成本相对直接后端增加约 **13%**。按 190 条指令外推，串行总运行时间约为 **38 小时**，API 总成本约为 **US$361**。

### 5. 专用方法、投稿后方法与 benchmark 区分能力

我们在共同支持的 joint-editing 任务上补充了专用音视频编辑方法 **AVI-Edit**，以及投稿后出现的 **Gemini Omni Flash** 和 **HappyHorse**。所有方法使用相同输入、checklist 和聚合规则。AVE-Agent 在 IF、Fidelity 和 Realism/AVQ 上分别达到 **69.09、63.64 和 49.56**，高于 AVI-Edit 的 **35.52、24.83 和 29.75**。同时，Gemini Omni Flash 在 AV Sync、Motion Smoothness 和 Video Fidelity 上领先，HappyHorse 在 Audio Aesthetic 上领先，AVE-Agent 在 IF、Realism/AVQ 和 Subject Consistency 上领先。这些跨指标排序反转表明 AVE-Compass 能同时识别新方法在底层质量上的进步和在复杂指令执行上的能力差异。

### 6. 开源 judge 与可复核的评测产物

为检验主要比较是否依赖单一闭源 judge，我们将主观评测模型替换为开源的 **Qwen3-Omni-30B-A3B-Instruct**，并在完整评测集上保持输入、checklist、评分维度和聚合方式不变。AVE-Agent 在 IF、Fidelity 和 Realism/AVQ 上均保持第一，三种方法的排序与 Gemini 3.1 Pro 一致。我们还将公开完整模型版本与推理配置、prompt 模板、生成后的 checklist、逐项判断缓存和聚合规则。固定中间产物保证已报告分数可精确追溯，开源 judge 的排序一致性则降低了重新运行对单一闭源模型的依赖。

### 7. 跨模态失败来源与 Figure 4 的操作性解释

我们将“joint editing 更困难”进一步分解为显式与隐式指令的配对分析。隐式指令下总体得分下降 **9.19 分**（95% CI [4.53, 13.95]），其中 IF 和 AV Quality/Sync 分别下降 **19.27 和 14.61 分**。仅描述音频变化、要求推断视觉后果时，总体下降为 **14.66 分**，高于反向推断的 **5.54 分**；Material/Texture Shift、Source Erasure 和 Identity Swap 是下降最集中的操作。这些结果将核心难点定位为从不完整指令中恢复对象—声源依赖，并在两个模态中同步执行删除、材质变化或身份替换。

针对 Figure 4，我们进一步给出 Content Difficulty 和 Instruction Semantic Complexity 的操作性定义。时长分析表明，随视频变长，Wan2.7 和 LTX2 出现退化，AVE-Agent 则保持稳定。这一趋势与结构化视频分析、Subtask DAG 和 Evaluator-driven correction 的设计相一致：长时序编辑被转化为可分解、可检查和可定向修正的子问题。

综合来看，新增结果形成了一条完整的证据链：五次重复评测与类别分解验证 benchmark 结果的稳定性和来源；1,173 个人工参考判断验证 checklist-guided MLLM 评测的可靠性；受控消融与效率分析分别说明 AVE-Agent 的性能来源和边际开销；专用方法、投稿后方法和开源 judge 则扩展了方法与评测后端的覆盖。这些分析共同支持 AVE-Compass 作为音视频联合编辑 benchmark 的稳定评测能力，以及 AVE-Agent 在复杂跨模态指令上的结构化编辑优势。
