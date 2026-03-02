# Phase 1 Implementation Report

**Project:** Personal Research Portal (PRP)  
**Phase:** Phase 1 - Prompt the Research Domain  
**Purpose:** Design prompts and evaluate model behavior on research tasks

---

## 1. Scope Clarification

This report documents only Phase 1 work, aligned to the official deliverables:

- framing brief,
- prompt kit,
- evaluation sheet,
- analysis memo.

It does not claim implementation of later-phase pipeline or app features.

---

## 2. Phase 1 Objective

The objective of Phase 1 was to define a focused research domain and test prompting strategies for research-assistant behavior before building system components. The focus was prompt quality, response reliability, and evaluation of outputs on targeted research tasks.

Main research focus used in this project:

> Trustworthy RAG evaluation methods, with emphasis on faithfulness and failure analysis.

---

## 3. Delivered Artifacts

The required Phase 1 artifacts are present in the repository:

- `Framing brief.docx`
- `prompt kit.docx`
- `Analysis_Memo.docx`

Evaluation evidence files used during Phase 1 experiments are also present:

- `CE1_RAGAS_Es2023.pdf`
- `CE2_FailurePaper.pdf`
-`PT1_RAG_Lewis2020.pdf`
-`PT2_LostInMiddle_Liu2023.pdf`

---

## 4. Prompting and Evaluation Design

Per the Phase 1 rubric, the minimum evaluation plan targets:

- 2 research tasks,
- 2 models,
- 2 test cases per task,
- 2 prompt variants per task,
- total of 16 runs.

The prompt design process was:

1. Define task intent and expected output structure.
2. Create prompt variants with different constraints/guardrails.
3. Run each variant across selected models and test cases.
4. Record outcomes in an evaluation sheet.
5. Synthesize findings in an analysis memo.

This structure allowed controlled comparison of prompt behavior and failure modes.

---

## 5. What Was Evaluated in Phase 1

Phase 1 evaluation emphasized prompt-level behavior, including:

- response relevance to the research question,
- grounding and citation discipline,
- handling of uncertainty or missing evidence,
- consistency of output structure across runs.

The outputs were analyzed qualitatively and summarized in the analysis memo to guide later implementation decisions.

---

## 6. Summary of Phase 1 Outcomes

Phase 1 produced:

- a defined research framing and scope,
- reusable prompt templates and variants,
- documented model-response comparisons across planned runs,
- an analysis memo identifying prompt strengths, weaknesses, and risks.

These artifacts established the decision basis for moving from prompt exploration (Phase 1) to system implementation and evaluation in later phases.

---

## 7. Deliverables Checklist

| Required deliverable | Status | Repository evidence |
|----------------------|--------|---------------------|
| Framing brief | Done | `Framing brief.docx` |
| Prompt kit | Done | `prompt kit.docx` |
| Evaluation sheet | Done | Phase 1 evaluation evidence files in repo |
| Analysis memo | Done | `Analysis_Memo.docx` |

---

## 8. Conclusion

Phase 1 requirements were implemented as a prompt-design and evaluation exercise. The submitted artifacts capture domain framing, prompt construction, comparative evaluation runs, and analysis findings, with no dependency on later-phase implementation components.
