# Organize Theme Taxonomy

Classification rules for routing collected entries into theme files.

---

## How Classification Works

For each entry, evaluate its `Title`, `Keywords`, and `Notes` fields against the signal keywords below. An entry may match more than one theme — append it to **all** matching theme files. If no theme matches, create a new theme file with an appropriate name (see "New Theme Creation" at the bottom).

---

## Theme: `llm-agents.md`

**Covers:** LLM-powered agents, multi-agent systems, long-horizon task planning, agent personalization, agent reliability, agentic workflows, agent training.

**Signal keywords:**
- agent, agentic, autonomous agent, multi-agent, AI agent
- long-horizon, long-horizon task, sequential decision
- reinforcement learning for agents, agent RL, PAHF, continual personalization
- canonical path deviation, agent failure, agent reliability
- planning, tool use, tool-calling, function calling
- emergent collective behavior, social systems with agents
- multi-agent RL, dynamic agents, variable agent count
- roadmap (when describing agent capabilities)

**Example entries from current data:**
- "Training LLM agents for extremely long-horizon tasks" (x-bookmarks)
- "Capable but Unreliable: Canonical Path Deviation" (alphaxiv-rec)
- "PAHF: continual personalization" (x-bookmarks)
- "New research from Google DeepMind — multi-agent learning algorithms" (x-bookmarks)
- "muratcankoylan/Agent-Skills-for-Context-Engineering" (github)

---

## Theme: `reasoning-and-knowledge.md`

**Covers:** Chain-of-thought reasoning, reasoning faithfulness, causal reasoning, knowledge editing, knowledge unlearning, long-tail knowledge, parametric knowledge, pretraining knowledge.

**Signal keywords:**
- chain-of-thought, CoT, efficient reasoning, reasoning faithfulness
- counterfactual, causal reasoning, causal judgment
- knowledge editing, knowledge unlearning, machine unlearning, KUDA, DUET, COIN
- parametric knowledge, NanoKnow, pretraining corpus
- long-tail knowledge, knowledge taxonomy
- hallucination detection (when framed as a knowledge/reasoning problem)
- representation deviation, context reliance

**Example entries from current data:**
- "The Art of Efficient Reasoning" (alphaxiv-rec)
- "NanoKnow: How to Know What Your Language Model Knows" (alphaxiv-rec)
- "CausalFlip: A Benchmark for LLM Causal Judgment" (alphaxiv-rec)
- "Anatomy of Unlearning" (alphaxiv-rec)
- "KUDA: Knowledge Unlearning by Deviating Representation" (alphaxiv-rec)

---

## Theme: `evaluation-and-benchmarks.md`

**Covers:** LLM evaluation methodology, benchmark saturation, LLM-as-judge, adversarial robustness, benchmark design, p-hacking in AI, scoring reliability.

**Signal keywords:**
- benchmark, benchmarking, benchmark saturation
- LLM-as-judge, judge, overrating, relevance assessment
- adversarial robustness, lexical sensitivity, syntactic sensitivity
- p-hacking, scientific integrity, AI writing papers
- evaluation, scoring, performance evaluation
- long-context evaluation, code QA evaluation
- RFEval, CausalFlip (as benchmark paper), NanoKnow (as benchmark paper)

**Example entries from current data:**
- "When AI Benchmarks Plateau: Benchmark Saturation" (alphaxiv-rec)
- "When LLM Judges Inflate Scores" (alphaxiv-rec)
- "Same Meaning, Different Scores: Lexical and Syntactic Sensitivity" (alphaxiv-rec)
- "AI is about to write thousands of papers. Will it p-hack them?" (x-bookmarks)
- "RFEval: Benchmarking Reasoning Faithfulness" (alphaxiv-rec)

---

## Theme: `ai-safety.md`

**Covers:** AI alignment, safety fine-tuning, safety erosion, red teaming, mental health AI risks, value alignment, value entanglement, harmful content, misalignment.

**Signal keywords:**
- alignment, safety alignment, misalignment
- safety fine-tuning, fine-tuning erosion, narrow fine-tuning
- red teaming, safety gaps, harmful data
- mental health AI, AI psychotherapy, patient simulation
- value entanglement, moral reasoning, moral values
- vision-language safety, multimodal safety
- hallucination (when framed as a safety risk)

**Example entries from current data:**
- "Assessing Risks of Large Language Models in Mental Health Support" (alphaxiv-rec)
- "Narrow fine-tuning erodes safety alignment in vision-language agents" (alphaxiv-rec)
- "Value Entanglement: Conflation Between Different Kinds of Good" (alphaxiv-rec)

---

## Theme: `tools-and-infra.md`

**Covers:** GitHub repos (tools, frameworks, utilities), browser extensions, LLM pretraining data pipelines, HTML extraction, data curation infrastructure, developer tools.

**Signal keywords:**
- github repo, repository, tool, framework, library, utility
- browser extension, bookmark sync, X tools, Twitter tools
- HTML extraction, HTML-to-text, pretraining data, data curation
- workflow, planning tool, markdown workflow, Claude skills
- NLP interface, spacecraft, heliophysics
- curated list, awesome list, resources compilation

**Classification rule for repos:** All GitHub repos go into `tools-and-infra.md` by default **plus** any additional topic theme if the repo's description strongly matches that theme's signals (e.g., an agent-focused repo also goes into `llm-agents.md`).

**Example entries from current data:**
- "baryon/x-copilot" — browser extension for X bookmarks (github)
- "OthmanAdi/planning-with-files" — markdown planning workflow (github)
- "hridaydutta123/awesome-twitter-tools" (github)
- "Beyond a Single Extractor: HTML-to-Text Extraction" (alphaxiv-rec)
- "muratcankoylan/Agent-Skills-for-Context-Engineering" (github) — also llm-agents

---

## New Theme Creation

When an entry matches **none** of the above themes:

1. Identify the dominant concept (e.g., "synthetic data", "heliophysics", "bespoke software").
2. Create a new file `./data/organize/<concept-slug>.md` using the table schema from `schema.md`.
3. Add a one-line comment at the top of the new file: `<!-- Created by /organize on <date>: <reason> -->`
4. Continue classifying remaining entries — subsequent entries with the same concept go into the new file.

**Naming convention:** lowercase, hyphen-separated, descriptive slug (e.g., `synthetic-data.md`, `bespoke-software.md`).

---

## Ambiguous Cases

- **Continual learning**: route to `reasoning-and-knowledge.md` if focus is knowledge retention/forgetting; to `llm-agents.md` if focus is agent adaptation; to both if both aspects are prominent.
- **Hallucination detection**: route to `reasoning-and-knowledge.md` unless framed primarily as a safety issue, in which case also add to `ai-safety.md`.
- **Synthetic data / social data** (e.g., "Next Reply Prediction X Dataset"): if no existing theme fits, create `synthetic-data.md`.
- **Bespoke software / AI coding** (e.g., Karpathy's bespoke software tweet): create `ai-software-development.md`.
