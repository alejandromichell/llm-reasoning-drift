# LLM Reasoning Drift Analysis

**Evidence Left Behind: Reasoning Drift and Silent Execution in a Single-Agent LLM Development Session**

*Alejandro Michell, Rogue Assembly (2026)*

---

## Overview

This repository contains the analysis pipeline for a study of reasoning drift in a real-world Claude Code session spanning 20 calendar days and approximately 62 hours of active development work. The session involved production incident response, AWS infrastructure, and a JavaScript module loading bug investigation. All within the same persistent agent context.

The pipeline annotates 1,347 assistant speaking turns using a six-label taxonomy (GROUNDED, INFERRED, ASSERTED, CORRECTING, ACTION, MIXED), applies eight ML and statistical methods, and integrates 915 silent action turns into the full 2,262-turn sequence to reveal what speaking-turns-only analysis misses.

**Preprint:** forthcoming on arXiv
**Medium article:** forthcoming

---

## Key Findings

- **61.3% of unsupported assertions immediately triggered tool execution** in the full sequence, nearly doubled from the 35.5% visible in speaking turns alone
- **SILENT_BASH was the single most common event type** at 30.6% of all turns, exceeding GROUNDED reasoning (22.6%). The session that appeared to be "an AI reasoning about code" was, in its event distribution, primarily an AI silently running shell commands
- **Median drift velocity was 4.0 steps** from last grounded evidence to each assertion, too low for threshold-based early warning, requiring a gate at the point of execution rather than monitoring for long drift sequences
- **Human oversight was present at only 21.8% of decision points**, 492 human messages out of 2,262 assistant turns. The grounding gate cannot rely on human attention; it must be built into the agent's pre-execution logic

---

## Repository Structure

```
llm-reasoning-drift/
├── notebook/
│   └── reasoning_drift_analysis.ipynb   # Full pipeline, outputs cleared
├── figures/
│   ├── fig1_reasoning_quality_timeline.png
│   ├── fig2_transition_matrix_heatmap.png
│   ├── fig3_hmm_hidden_states.png
│   ├── fig4_drift_velocity_speaking_only.png
│   ├── fig5_drift_velocity_full_sequence.png
│   └── fig6_drift_velocity_slope_chart.png
└── data/
    └── annotation_schema.json           # Dataset field definitions — no raw data
```

---

## How to Run

The notebook runs in [Google Colab](https://colab.research.google.com/) against your own Claude Code session transcript.

**Requirements:**
- A Google account with Google Drive
- A Gemini API key (paid tier recommended, pipeline runs ~1,347 annotation calls)
- Your Claude Code session JSONL transcript uploaded to Google Drive

**Setup:**

1. Open `notebook/reasoning_drift_analysis.ipynb` in Google Colab
2. Add your Gemini API key as a Colab secret named `llm-reasoning-colab-analysis-gemini`
   - Left sidebar -> Key icon -> Add new secret
3. Upload your Claude Code transcript to Google Drive
   - Claude Code transcripts are stored locally at:
     `~/.claude/projects/<project-path>/<session-id>.jsonl`
4. Update `TRANSCRIPT_PATH` in Cell 4 to point to your uploaded file
5. Run cells in order: **Cell 4 -> Cell 6 -> Cell 10 -> Cell 16 -> onward**

**Note:** Cell 34 (Thinking Block Analysis) cannot execute, Claude Code stores thinking blocks as cryptographic signatures only, not plaintext. The `had_thinking` binary flag (Cells 27–28) remains valid for presence detection.

---

## Pipeline

Eight ML and statistical methods applied in sequence:

| Cell(s) | Method | Purpose |
|---|---|---|
| 10–16 | LLM annotation (Gemini) | Label all 1,347 speaking turns with six-category taxonomy |
| 17 | Transition matrix | 6×6 probability map of label-to-label transitions |
| 19–20 | Hidden Markov Model (3 states) | Discover latent behavioral states in the label sequence |
| 29 | Fisher's exact test / chi-square | Validate Extended Thinking ASSERTED rate difference |
| 30 | Sentence embeddings + KMeans (k=4) | Unsupervised functional taxonomy of ASSERTED messages |
| 31 | Drift velocity (speaking turns) | Steps since last GROUNDED before each ASSERTED event |
| 32 | Full-sequence transition modeling | Integrate 915 silent turns into the complete 2,262-turn sequence |
| 33 | Evidence gap analysis (Gemini) | Identify specific unsupported claims in each ASSERTED turn |

Additional cells: SILENT_BASH origin split (Cell 35), full turn origin split (Cell 37).

---

## Data

Raw transcript and annotated datasets are not included in this repository, the session contains project-specific development context.

`data/annotation_schema.json` documents the field definitions for the annotated dataset so the pipeline output format is reproducible against any Claude Code session.

---

## Citation

```
@misc{michell2026evidence,
  title   = {Evidence Left Behind: Reasoning Drift and Silent Execution
             in a Single-Agent LLM Development Session},
  author  = {Michell, Alejandro},
  year    = {2026},
  note    = {Preprint. Rogue Assembly.}
}
```

---

## License

MIT
