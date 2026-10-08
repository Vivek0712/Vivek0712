<div align="center">

# Vivek Raja P S

**I build AI agents that have to survive production, and I publish the measurements.**

Isolated execution on Firecracker microVMs. Decision models that answer in 100 ms with calibrated confidence. Governed memory. Robots that take orders in plain English. Everything below links to a repo, a benchmark run, or a merged PR.

<p>
  <img src="https://img.shields.io/badge/AWS_AI_Hero-since_2021-FF9900?style=flat-square&logo=amazonaws&logoColor=white" />
  <img src="https://img.shields.io/badge/Microsoft_MVP-AI_%26_ML-5E5E5E?style=flat-square&logo=microsoft&logoColor=white" />
  <img src="https://img.shields.io/badge/Senior_AI_Solutions_Architect-Aivar_Innovations-0A66C2?style=flat-square" />
  <img src="https://img.shields.io/badge/Lead-AWS_UG_Madurai-232F3E?style=flat-square" />
</p>

<p>
  <a href="https://www.linkedin.com/in/vivekrajaps/"><img src="https://img.shields.io/badge/LinkedIn-vivekrajaps-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white" /></a>
  <a href="https://twitter.com/meetvivekraja"><img src="https://img.shields.io/badge/X-@meetvivekraja-000000?style=for-the-badge&logo=x&logoColor=white" /></a>
  <a href="https://www.youtube.com/@vivekrajaps"><img src="https://img.shields.io/badge/YouTube-@vivekrajaps-FF0000?style=for-the-badge&logo=youtube&logoColor=white" /></a>
  <a href="https://aws.amazon.com/developer/community/heroes/vivek-raja-p-s/"><img src="https://img.shields.io/badge/AWS_Hero-profile-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white" /></a>
</p>

</div>

<table align="center">
<tr>
<td width="33%" valign="top">

### Hire

Agentic AI on Bedrock and AgentCore, migrations to Bedrock (OpenAI, Azure OpenAI, Vertex), retrieval over large corpora, Amazon Q / Quick. Architect who ships the code and the benchmark.

[Talk on LinkedIn ->](https://www.linkedin.com/in/vivekrajaps/)

</td>
<td width="33%" valign="top">

### Contribute

Open an issue first, then a PR. Most useful right now:
[microvm-ctl](https://github.com/Vivek0712/microvm-ctl) (runtimes, fleet scenarios)
[decider-lab](https://github.com/Vivek0712/decider-lab) (adapters, eval sets)
[memwarden](https://github.com/Vivek0712/memwarden) (poisoning detectors)

</td>
<td width="33%" valign="top">

### Reach out

Talks, workshops, AWS community events, or a question about anything here.

vivekraja98 [at] gmail [dot] com
Bengaluru / Madurai, India

</td>
</tr>
</table>

<br/>

<div align="center">

| 3.54 s | 111 ms | 93.8% | 57,796/s | 0.7 s | +23.4 |
|:---:|:---:|:---:|:---:|:---:|:---:|
| microVM launch to serving traffic (p50) | warm request inside the VM (p50) | cheaper than always-on for a 30 min + 8 h session | governed memory writes per second, 0 poisoned records admitted | first request wakes a suspended VM | Intelligence gain from a 150-step decider fine-tune, 95% CI [18.2, 28.7] |

<sub>Every number is from a recorded run in the linked repos, not a quote from a datasheet.</sub>

</div>

<br/>

## 1. Agents inside AWS Lambda MicroVMs

Each agent gets its own Firecracker VM and kernel. [Strands Box](https://github.com/strands-agents/box) runs inside and checks every shell command, file touch and outbound request against a policy. I wrote the control plane, the fleet runtime, the playground and the benchmarks.

<table>
<tr>
<td width="50%" valign="top">

[**microvm-ctl**](https://github.com/Vivek0712/microvm-ctl) · [PyPI](https://pypi.org/project/microvm-ctl/) · Apache-2.0
Build a snapshot image from a Dockerfile, launch Firecracker microVMs, scale fleets at 80% of applied quota, call each VM over an authenticated endpoint, hand tasks over via a lease from Step Functions or Lambda durable functions.

| What | Measured (us-east-1) |
|---|---|
| Dockerfile to runnable snapshot | 123 to 145 s |
| RunMicrovm to serving authenticated traffic | p50 3.54 s, p95 4.49 s |
| Warm request, real Python inside the VM | p50 111 ms |
| Suspend / resume, same PID, state intact | 2.5 s / 2.6 s |
| Fleet 0 to 6 VMs on a 1 launch/s quota | 9.7 s wall |
| 30 min active + 8 h suspended vs always-on | 93.8% cheaper |

</td>
<td width="50%" valign="top">

<img src="https://raw.githubusercontent.com/Vivek0712/microvm-ctl/main/docs/img/benchmark.png" alt="mvm benchmark transcript" />
<sub>The benchmark run that produced the table. Harness in <code>benchmarks/benchmark.py</code>.</sub>

</td>
</tr>
<tr>
<td width="50%" valign="top">

<img src="https://raw.githubusercontent.com/Vivek0712/strands-box-microvm-ctl/main/media/08-fanout.png" alt="Strands Box playground fan-out" />
<sub>48 policy-checked boxes dispatched over 4 warm VMs in 4.08 s: 528 policy decisions, 96 denials, 48 of 48 correct.</sub>

</td>
<td width="50%" valign="top">

[**strands-box-microvm-ctl**](https://github.com/Vivek0712/strands-box-microvm-ctl) · [live playground](https://d27duseaq87rqu.cloudfront.net)
Strands Box built for aarch64 Linux, one box per task, fleet scenarios, and a playground behind CloudFront + WAF. Also contributed Linux support upstream to Strands Box.

| What | Measured |
|---|---|
| One box inside a warm VM (create, run, 26 decisions, tear down) | 83 ms p50 |
| Boxes per second in one VM | 22.7, flat to 32 concurrent, 32/32 correct |
| 8 VMs x 8 boxes, per round | 64/64 correct, 1,664 decisions, 192 denials |
| 96 tasks over 4 VMs | 14.9 tasks/s at 8 in flight, 96/96 correct |
| Strands agent on Nova 2 Lite inside a VM | 2.7 to 3.7 s per task warm; `.env` read and delete denied and quoted back |

</td>
</tr>
<tr>
<td width="50%" valign="top">

[**microvm-handoff-demo**](https://github.com/Vivek0712/microvm-handoff-demo)
Nine scenarios run live on both Step Functions and Lambda durable functions, with execution histories, VM logs and console screenshots checked in. An 8-shard fan-out completes in 8.9 s for $0.00031 per run. The first pass found two bugs that became microvm-ctl 0.3.1.

[**awesome-microvm**](https://github.com/Vivek0712/awesome-microvm)
14 runnable examples (code sandbox, agent eval fan-out, notebook sessions, DuckDB over S3, CI runner, multi-tenant agents, circuit breaker) and a 5-part article series on AWS Builder Center.

</td>
<td width="50%" valign="top">

<img src="https://raw.githubusercontent.com/Vivek0712/microvm-handoff-demo/main/benchmarks/results/bench.png" alt="handoff benchmark" />
<sub>p50 end-to-end by orchestrator, 18 runs, 1 / 4 / 8 shards.</sub>

</td>
</tr>
</table>

Articles: [Control and scale Lambda MicroVMs with microvm-ctl](https://builder.aws.com/content/3JIDTpz0ZgatSBv24drra3gEod9/control-and-scale-aws-lambda-microvms-with-microvm-ctl) · [Seven workloads Lambda could never run until MicroVMs](https://builder.aws.com/content/3JJ2oNWY9EsZzivMMx044cSlrFQ/seven-workloads-lambda-could-never-run-until-microvms) · [A kernel for every customer: 1,000 tenants](https://builder.aws.com/content/3JJ7tASPSSUUTrcpWnWtxOuu8g3/a-kernel-for-every-customer-scaling-ai-agents-to-1000-tenants-on-aws-lambda-microvms-with-microvm-ctl) · [Hand a task to a MicroVM from anywhere](https://builder.aws.com/content/3Jcx0VcEOq9QQBtqr59y4S8dVFC/hand-a-task-to-a-microvm-from-anywhere-one-lease-step-functions-durable-functions-or-your-own-controller)

<br/>

## 2. Training and shipping decision models

[Strands Decider](https://github.com/strands-labs/strands-decider) is a 2B "system one" model: one forward pass, no decoding, a calibrated probability on every yes/no, choice or score question, ~115 ms on a 3090. I work on making it see, making it better, and making it deployable.

<table>
<tr>
<td width="50%" valign="top">

**Giving the decider eyes, with zero image training**
Qwen3.5-2B-Base ships a vision tower nobody was using. I remapped v19's LoRA onto the multimodal torso and put images inside `<state>`. Merged upstream as [strands-decider PR #10](https://github.com/strands-labs/strands-decider).

| System | NaturalBench acc | NB ECE | POPE-adv acc |
|---|---|---|---|
| Qwen3.5-2B-Base, untrained readout | 0.712 | 0.048 | 0.865 |
| **v19 `--vision`, no image training** | **0.784** | **0.014** | **0.877** |
| Mapika/decider-2b-vision, image-trained | 0.794 | 0.080 | 0.860 |

Matches an image-trained 2B on accuracy (paired bootstrap CIs cross zero), with 5x lower calibration error. Cost of the whole eval: one H100 for 35 minutes, $1.58.

**Then training it on 38k image rows** (VQAv2 pairs, COCO counting, Mind2Web, own chart and invoice renders), 3 seeds, ViT frozen, ~56 min per run:

| | NB acc | POPE acc | POPE ECE | blind-image confidence |
|---|---|---|---|---|
| v19 | 0.787 | 0.858 | 0.082 | 0.652 |
| **fine-tuned (A)** | **0.805** | 0.874 | **0.032** | **0.582** |

Better on every image benchmark and every seed, no regression on 899 held-out text rows (0.817 -> 0.829). The blind-image column matters: before training, the model stayed confident with the image removed. Now it hedges.

</td>
<td width="50%" valign="top">

**Training better text deciders**
Base bake-off at 2,400 steps: MiniCPM5-2B beats Qwen3.5-2B by +13.0 JevBench tasks (CI +6.3, +20.0). My v21 candidate on MiniCPM5 scores 179/231 on the public set vs the published v19's 167, a paired gain of +12.3 tasks (CI +3.0, +22.3).

The honest part: on 1,050 rows from five families no decider had seen, MiniCPM5's lead evaporates (-17 to -21) and a Qwen weight soup wins. Every model I trained still beats the two competing open 2B deciders on those unseen families (+2.6 to +14.5). All of this is preregistered, three seeds per arm, and labelled exploratory until a confirmatory run. [Full results](https://github.com/Vivek0712/strands-decider/blob/b63bc26de24a935a4925ed96616954f82daf1f80/research/v21/RESULTS.md)

<img src="https://raw.githubusercontent.com/Vivek0712/decider-lab/main/docs/img/results.png" alt="decider-lab results" />
<sub><a href="https://github.com/Vivek0712/decider-lab">decider-lab</a>: evaluate, calibrate, compare and fine-tune any decision model from one YAML, on a laptop, over ssh, on EC2 or a rented GPU. This whole lab ran in 537 s on one A100.</sub>

<img src="https://raw.githubusercontent.com/Vivek0712/strands-decider-agentcore/main/docs/img/triage-console.gif" alt="triage console on AgentCore" />
<sub><a href="https://github.com/Vivek0712/strands-decider-agentcore">strands-decider-agentcore</a>: the official v21 as a session-isolated endpoint on Bedrock AgentCore Runtime, int8 on 2 vCPU. Cold start 45 to 90 s, 4.2 GB resident, int8 vs local 30/30 identical decisions. Here it routes support tickets; 4 of 8 cleared without human review.</sub>

</td>
</tr>
</table>

<br/>

## 3. Robots that take orders in English

<table>
<tr>
<td width="50%" valign="top">

<img src="https://raw.githubusercontent.com/Vivek0712/strands-robots-ros2/main/assets/curiosity_rover.gif" alt="Curiosity rover driven by an Amazon Nova agent" />
<sub><a href="https://github.com/Vivek0712/strands-robots-ros2">strands-robots-ros2</a>: an Amazon Nova agent driving a NASA Curiosity rover in Gazebo over rosbridge. Four runtimes (rosbridge, ros2, http, deepracer) behind one adapter; the ros2 runtime adds Nav2 planning and live SLAM.</sub>

</td>
<td width="50%" valign="top">

<img src="https://raw.githubusercontent.com/Vivek0712/strands-agentic-deepracer/main/assets/phase3gif.gif" alt="DeepRacer closed-loop vision navigation" />
<sub><a href="https://github.com/Vivek0712/strands-agentic-deepracer">strands-agentic-deepracer</a>: Nova Lite plans, Nova Pro watches the front camera between steps and decides continue / replan / abort. 15 verified navigation patterns, vision call 1 to 3 s with a 4 s safe fallback.</sub>

</td>
</tr>
</table>

**Upstream to [strands-labs/robots](https://github.com/strands-labs/robots)** (150+ robots, 8 categories): 9 merged PRs authored, 3 co-authored. The ones I care about most: rosbridge transport with `RosbridgeRobot.from_curiosity()` ([#1110](https://github.com/strands-labs/robots/pull/1110)), ROS 2 action support and goal-level `navigate_to` ([#995](https://github.com/strands-labs/robots/pull/995)), AWS DeepRacer as an Ackermann ROS 2 robot ([#2689](https://github.com/strands-labs/robots/pull/2689)), FastTD3 as a native off-policy RL provider ([#3206](https://github.com/strands-labs/robots/pull/3206)).

<br/>

## 4. Memory that cannot be poisoned quietly

[**memwarden**](https://github.com/Vivek0712/memwarden) wraps any agent memory backend (AgentCore Memory, Redis, pgvector) with tenant isolation, per-category retention, provenance-based trust tiers, two detection layers, four fail-closed read gates, a hash-chained audit log and GDPR Article 17 erasure certificates. Targets OWASP ASI06. Companion to the Engram paper.

Lab run, 6 tenants, 3,181 writes, 23% poisoned: **57,796 governed writes/s, 0.017 ms p50 overhead, 550 blocked at write, 188 quarantined, 0 adversarial records reached agent context.** The README says plainly that the inline detector is a precision instrument (held-out paraphrase recall 0/10) and the trust gate carries the guarantee (7 of 8 held-out attacks stopped at read).

<br/>

## 5. Agents that never compute a number

<table>
<tr>
<td width="55%" valign="top">

<img src="https://raw.githubusercontent.com/Vivek0712/f1-agentic-analysis/main/docs/img/dashboard.png" alt="F1 pit-wall dashboard" />

</td>
<td width="45%" valign="top">

[**f1-agentic-analysis**](https://github.com/Vivek0712/f1-agentic-analysis) · [live](https://pjjswsm6vf.us-east-1.awsapprunner.com)
Deterministic pandas/SciPy computes everything (Theil-Sen degradation fits, pit loss, undercut ledger). Seven Strands agents with `tools=[]` only write the brief, and a groundedness check rejects any number not in the payload. Frozen-clock backtest, 2024 Spanish GP: MAE 1.20 s at lap 20, 0.60 s at lap 40; Verstappen's first stop priced within 0.06 s of optimum.

[**assay**](https://github.com/Vivek0712/assay) · [live](https://d1m5fcrjmmi9ue.cloudfront.net)
Read-quality scoring for AWS Builder Center. Over 302 articles in one week, likes vs quality correlated at r = -0.09, and the top 20 by likes and top 20 by score shared zero articles. 96% exact verdict agreement with a hand-labelled golden set.

[**aws-samples / OpenClaw on AWS with Bedrock**](https://github.com/aws-samples/sample-OpenClaw-on-AWS-with-Bedrock) · ★ 415
Contributed the AgentCore Runtime deployment path ([PR #16](https://github.com/aws-samples/sample-OpenClaw-on-AWS-with-Bedrock/pull/16)), since superseded by the maintainers' multi-tenant build. Presented at ClawCon Seattle 2026. Also [clawskills-strands](https://github.com/Vivek0712/clawskills-strands): 52 OpenClaw skills converted to executable Strands tools.

</td>
</tr>
</table>

<br/>

## Research and background

**Papers.** Engram (memory governance, with memwarden), One Agent Many Bodies and Design Principles of Agent-Powered Robotics Systems (robotics, simulation-validated, arXiv pending). 7 peer-reviewed papers, 3 granted patents, across AI and brain-computer interfaces.

**Before Aivar.** Five years as VP of Products at Nexstem, a brain-computer interface startup: EEG hardware with motorized electrodes, firmware, signal pipelines, on-device inference, SDK. The path ran silicon -> firmware -> embedded -> cloud -> ML -> agents -> robots, which is why I care about what sits under the model.

**Community.** Founded AWS User Group Madurai; help run communities across Madurai, Chennai, Coimbatore and Bengaluru. 80+ talks including AWS re:Invent, 500+ meetups. AWS Machine Learning Hero at 22, now AWS AI Hero.

<br/>

<div align="center">

`Bedrock` `AgentCore` `Strands Agents` `Strands Decider` `Strands Box` `Firecracker` `Lambda MicroVMs` `ROS 2` `Nav2` `OpenSearch` `CDK` `Python` `TypeScript`

<sub>I occasionally build things because somebody tells me they probably shouldn't work.</sub>

</div>
