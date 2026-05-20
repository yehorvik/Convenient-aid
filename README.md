# Convenient Aid 
This project can be used for purposes of monitoring the needs of humanitarian aid among the population of a certain city.
To start a project, it is important to execute setup.sql beforehand, which will generate essential tables in your PostgreSQL database.
Also, it is recommended to view the possible [solution for the execution of pollings](https://github.com/Yilchy/Convenient-Aid-Bot) and [interface for representing results of polling](https://github.com/Yehorvik/react-VP)


# The Current State of Reinforcement Learning
 
---
 
## Why this matters now
 
The 2026 conversation about reinforcement learning has been dominated by one story: verifiable rewards in language and reasoning models — RLVR, GRPO, the o-series and DeepSeek-R1, the long tail of agentic tool-use. That story is real and it is consequential. It is also incomplete.
 
Running in parallel is a second story the discourse has underweighted: verifiable *costs* in stochastic dynamic problems where the reward function isn't learned at all, where modern function approximation lets classical operations-research formulations scale to problems they couldn't reach before, and where the environment is built from real-world signal rather than fabricated simulation. The app I'll present sits in the second story. This brief is about why that story deserves attention from people working in the first.
 
---
 
## The first story, briefly: RLVR and reasoning RL
 
The 2022–2023 RLHF stack — reward model trained on preference pairs, PPO against the reward model, KL penalty against a reference policy — remains part of the production stack at the major labs, though increasingly alongside or downstream of RLVR-style training.
 
The pivotal move was reinforcement learning with verifiable rewards, brought into the mainstream by OpenAI's o1 and validated at scale by DeepSeek-R1 (Guo et al., *Nature* 2025) and the GRPO algorithm it popularized. Instead of training a reward model to approximate human preferences, RLVR trains directly against automatically checkable signals: unit tests pass or fail, a math answer matches the ground truth, a SQL query returns the expected rows. GRPO further simplified the algorithmic side — dropping the value network entirely and estimating advantage by comparing rollouts within a group on the same prompt. Less infrastructure, fewer moving parts, comparable or better results on reasoning tasks.
 
This worked far better than most of the field expected. At IMO 2025, OpenAI's experimental reasoning model and Google DeepMind's Gemini Deep Think each scored 35 out of 42, solving five of six problems under standard contest conditions — the gold-medal threshold, achieved for the first time by general-purpose reasoning models rather than specialized formal-proof systems.
 
One way to read the RLVR shift is as a rediscovery: a corner of the field that had been working with learned reward models for years noticed that, where you can write down the reward, the algorithmic problem becomes radically easier and the engineering problem moves elsewhere. This was not news to operations research. Classical OR has had verifiable cost functions for decades — transport costs, holding costs, deprivation costs in humanitarian logistics, makespan in scheduling — and the bottleneck in OR was never the reward signal but the curse of dimensionality. The interesting development of 2025 is not that RLVR exists. It's that two communities — the LLM-RL community arriving at verifiability from above, and the OR-RL community arriving at function approximation from below — have ended up using structurally similar techniques to attack structurally similar problems. The brief argues that this convergence is the actual headline.
 
---
 
## Environments, fabricated and signal-driven
 
In September 2025, The Information reported that Anthropic had discussed spending over $1 billion on RL environments over the following year (subsequently consolidated and contextualized in Epoch AI's January 2026 industry survey of RL environment builders). The economics are real: contract sizes are often six to seven figures per quarter, with task costs typically in the $200–$2,000 range and high-fidelity environment replicas of complex products like Slack reaching around $300k.
 
Quality criteria are non-obvious and the field has converged on them empirically. Reward-hacking robustness is cited as the most important: high reward must mean the task was actually solved, not gamed, and creating robust graders takes many iterations. Difficulty calibration matters almost as much — tasks need a pass rate above 0% and below 100% to provide a learning signal, with practitioners typically targeting a minimum of 2–3% pass rate, or at least one success in 64–128 rollouts. Tasks become less useful once their pass rate climbs into the high range, and are rotated out of training. The skills required are also not the ones you might guess: building environments is mostly engineering, but creating good tasks demands domain knowledge and expert-level prompting more than ML chops — the best task authors tend to be heavy frontier-model users and benchmark designers, not necessarily AI researchers.
 
The discourse around RL environments has settled around one image: fabricated simulators — Slack workspaces rebuilt at fidelity, browser sandboxes, IDE replicas, enterprise-SaaS task farms. This image captures most of the dollars but not all of the design space. A second pattern is worth naming: *signal-driven environments*, where the structure of the environment is not engineered from scratch but assembled from real-world data. The clearest analogy from the LLM-RL side is the RLHF preference dataset — human raters' choices shape what the reward model rewards, and the reward model in turn shapes what the policy learns. The dataset *is* a structural component of the environment, not an input to it. The same pattern appears, with different mechanics, in problems where real-world signal — usage data, sensor data, monitoring data, location data, votes — determines the geometry of the state and action space that the optimizer subsequently operates over. Fabricated environments are easier to talk about because their economics are legible. Signal-driven environments are harder to enumerate but at least as consequential, and they are where some of the most interesting applied RL is being deployed today.
 
---
 
## What's actually working in production
 
Stripping out the hype, here is what RL is reliably delivering in 2026.
 
**Operations-research-style stochastic optimization.** Routing, scheduling, datacenter cooling, chip placement, humanitarian logistics, inventory allocation, electricity grid dispatch. These are not new applications of RL — Powell's *Approximate Dynamic Programming* was published in 2007, and the underlying ideas trace back to Bellman and the neuro-dynamic-programming work of Bertsekas and Tsitsiklis — but they are among the most reliably productionized RL applications outside of LLM training, with decades of deployment evidence behind them. They share a structural property worth naming: the cost function is engineered, not learned. Transport cost is known. Deprivation cost can be specified. Wirelength on a chip is measurable. Power consumption in a datacenter is metered. The role of modern function approximation is not to estimate the reward but to estimate the value of being in a state — the future cost-to-go — when the state space is too large to enumerate. Google's AlphaChip (Mirhoseini et al., *Nature* 2021) used a learned value network to do this for chip placement; humanitarian-logistics work has used both linear function approximation, decomposed by district, and neural-network value functions over the same problem class (van Steenbergen, van Heeswijk, Mes, 2023). The interesting frontier is not whether these problems can be solved with RL — they can — but how value-function approximation choices interact with the structure of the cost surface, and how robust the resulting policies are to misspecification in the upstream signal that defines the problem.
 
**Reasoning capability in LLMs.** The o-series, DeepSeek-R1, Gemini Deep Think, and their successors are all products of RL on verifiable tasks. Reasoning-capability training has become the dominant single use of RL compute at frontier labs.
 
**Tool use and agentic behavior.** Long-horizon tool calling — search, code execution, browser control, MCP integrations — is being elicited primarily through RL, often with interleaved supervised fine-tuning on gold trajectories where good ones can be obtained cheaply.
 
**Coding assistants.** Beyond SWE-bench-style tasks, coding RL has moved toward productionized environments that include the IDE, git, issue trackers, and review workflows.
 
**Personalization, ranking, and recommendation.** The classical contextual bandit and off-policy RL stack continues to power ads, feeds, and recommendation systems at scale. Quietly mature; not the loud part of the field.
 
What is *not* yet reliably working: general robotic manipulation in open environments, sample-efficient continuous control without heavy simulation, anything requiring genuine multi-agent coordination at scale, and RL on goals that resist clean specification.
 
---
 
## Persistent problems, shared across the two stories
 
The hard problems in RL look superficially different across the two stories but share a common structure once stated carefully.
 
Reward hacking in RLVR has its analog in cost-function misspecification in OR-RL: in both cases, optimizing the proxy is not the same as optimizing the underlying goal, and in both cases the field's defenses are iteration, adversarial evaluation, and humility about what the metric is actually measuring. Reward hacking remains a chronic concern in RLVR specifically — METR's June 2025 analysis "Recent Frontier Models Are Reward Hacking" documents that the problem has not gone away as models have scaled; detection has improved faster than prevention. Cost-function misspecification in OR-RL is the older form of the same family of problems and is handled with similar tools: adversarial testing of the cost function against expert intuition, sensitivity analysis, and iterative refinement against deployment outcomes.
 
Sample efficiency is poor in LLM-RL because rollouts against fabricated environments are expensive in compute; sample efficiency is poor in OR-RL because rollouts against real-world signal are expensive in time and data, and the consequences of a bad rollout are not "wasted GPU hours" but decisions that affect actual people.
 
Evaluation is hard in both: in LLM-RL because the proxies (benchmarks) drift from the underlying capability they were meant to measure; in OR-RL because real-world deployment surfaces failure modes that the simulator-trained policy didn't anticipate.
 
The honest framing is not that one paradigm has solved these problems and the other hasn't. It is that both face them, with different mechanics, and recognizing the parallel structure is more useful than the disciplinary partition that put one of these traditions in the footnote.
 
---
 
## What this means for applied work: two playbooks
 
Two playbooks now coexist in applied RL, and conflating them produces bad advice in both directions.
 
**The LLM-RL playbook** starts from a strong pretrained base, frames training as eliciting capability the base model already has latent capacity for, builds environments and graders rather than reward models where possible, scales rollouts aggressively when verifiable signal is available, and treats reward hacking as a chronic adversarial problem. Almost no one trains language or perception policies from scratch.
 
**The OR-RL playbook** starts from a problem-structured cost function — engineered, not learned — defines a Markov decision process over a state space that is typically too large to enumerate, approximates the value function (linearly, by decomposition, or with neural networks), uses a combinatorial solver such as mixed-integer programming for the per-step decision, and warm-starts training with a heuristic policy to avoid converging on the local minima that random initialization produces. The "pretrained base" plays no role; the cost function plays the role the base model plays in LLM-RL — it is the structural prior that makes the problem tractable.
 
The playbooks converge on a deeper claim: where the signal is engineered, RL works. Where the signal has to be learned, RL struggles. This is among the most important practical insights to come out of the last three years, and it is paradigm-agnostic.
 
They diverge on what scaling means and where the engineering effort goes. LLM-RL scales by spending more compute on rollouts in a fabricated environment. OR-RL scales by improving the function approximation and by extracting better signal from the real-world data that defines the problem. The two are not competing for the same compute budget — they are competing for different kinds of engineering attention, and the field would benefit from talking about both as serious paradigms.
 
---
 
## What's underweighted in current discourse
 
Three threads the brief argues should be more central than they currently are: OR-style RL as a serious production paradigm beyond the usual ads-and-routing bullet; signal-driven environment construction as a category distinct from fabricated environments; and value-function approximation as the underused tool in the modern RL toolkit, particularly the decomposed and structured variants that scale to high-dimensional resource-allocation problems without the engineering cost of neural value networks.
 
---
 
## Bottom line
 
The interesting questions in RL in 2026 are not "which algorithm." They are about the surrounding scaffolding — how to construct environments that produce useful signal, how to engineer cost functions that resist the optimizer's tendency to game them, how to choose function-approximation structures that match the problem's cost surface, and how to evaluate honestly when the proxy is the only thing the team can measure. The contention of this brief is that two communities — the one that arrived at these questions through RLHF and reasoning models, and the one that has been answering versions of them for decades under the operations-research banner — are now close enough in technique that the wall between them is mostly historical.
 
The app I'll present is one instance of the second community's playbook, applied to a problem the first community would find unfamiliar in domain but recognizable in structure: stochastic dynamic allocation under uncertainty, with a verifiable cost function, with the environment partly assembled from real-world signal, and with the function approximation chosen for scalability rather than expressiveness. The question I'd most like the room to bring is how my system handles the place where the algorithm meets the implementation: the warm-start that prevents local minima but biases the trajectory it converges to; the value-function decomposition that scales but loses some shared structure across districts; the upstream monitoring signal that defines the state space but does not come with calibration guarantees. Those are the design choices that decide whether the system is useful or merely runnable. The talk is about how they were made.
 
---
 
*Primary sources drawn on for this brief: Guo et al. (2025), *Nature* — DeepSeek-R1; Mirhoseini et al. (2021), *Nature* — AlphaChip; Powell (2011), *Approximate Dynamic Programming: Solving the Curses of Dimensionality*, 2nd ed., Wiley; Bertsekas & Tsitsiklis (1996), *Neuro-Dynamic Programming*, Athena Scientific; van Steenbergen, van Heeswijk & Mes (2023), "The Stochastic Dynamic Post-Disaster Inventory Allocation Problem with Trucks and UAVs"; Barber & Denain (Epoch AI, January 2026), "An FAQ on Reinforcement Learning Environments"; Von Arx, Chan & Barnes (METR, June 2025), "Recent Frontier Models Are Reward Hacking"; OpenAI and Google DeepMind IMO 2025 announcements (July 2025).
