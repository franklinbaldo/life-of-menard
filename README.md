# life-of-menard 📜🗡️

_A branch-and-duel experiment that tries to “grow” **Don Quijote** out of Pierre Menard’s diary—powered by the open-weight **Hrönir Encyclopedia** engine._

---

## 1 Why does this repo exist?

> **Ultimate objective**  
> Discover—through branching biographies, duels, and OpenSkill ratings—one deterministic sequence of Pierre Menard’s diary fragments that, when concatenated with locked slices of the authentic Spanish **_Don Quijote_**, makes a temperature-0 open-weight LLM reproduce the novel **letter-for-letter**.

Workflow in a nutshell

1. Contributors craft diary fragments (_hrönirs_) about Menard’s successive lives.
2. Competing biographies (paths) duel for each position; votes + OpenSkill decide winners; the temporal cascade selects a canonical path.
3. Every even position is auto-filled with the next 2 000-character slice of the ground-truth _Quijote_ text.
4. After each cascade we feed the full canon (odd diary + even Quijote) to a zero-temperature **open-weight** LLM and store the SHA-256 of its output.
5. When that SHA equals the reference edition’s SHA, the project is complete.

Position 0 is immutable (“Menard’s manifesto”). Voting on pos 0 is rejected.

---

## 2 Project status

| Phase                    | Progress                                     |
| ------------------------ | -------------------------------------------- |
| **P0** base engine       | ✅ Hrönir core (paths • duels • OpenSkill)   |
| **P1** Menard extensions | ✅ ref-text interlock, CLI guards, SHA tests |
| **P2** community launch  | ⏳ first duel tournament                     |
| **Finish line**          | LLM output SHA = reference SHA 🚀            |

---

## 3 Repo layout

```text
life-of-menard/
├─ hronir_encyclopedia/      ← upstream engine
├─ menard_ext/
│  ├─ cli.py                 ← append-life, cast-votes, test-quixote
│  ├─ interlock.py           ← fills even slots with Quijote slices
│  └─ llm_backend.py         ← open-weight inference wrapper
├─ data/
│  └─ quijote_ref.txt        ← canonical Spanish text (UTF-8)
├─ docs/
│  ├─ ledger_protocol.md
│  └─ flow.svg
└─ tests/
```

---

## 4 Prerequisites

```bash
python -m pip install -r requirements.txt
```

- Python 3.10+
- DuckDB ≥ 0.10
- `openskill` 1.x
- Typer, Pydantic, Rich
- **Open-weight** LLM endpoint (e.g. Llama 3, Mistral, Phi-3) capable of deterministic generation (`temperature=0`)

---

## 5 Quick start

```bash
# 1. Initialise DB and position-0 stub
hronir init-test

# 2. Add first diary fragment (position 1)
hronir append-life --text "Pierre Menard, aged 22, joins a symbolist circle…"

# 3. View open duels
hronir list-pending-duels

# 4. Cast votes (token at position 4 ⇒ ceil(sqrt(4)) = 2 votes)
hronir cast-votes \
      --voting-token 0123abcd-… \
      --verdicts '{"3":"path_uuid_A","5":"path_uuid_B"}'

# 5. Run deterministic Quijote test
hronir test-quixote
```

Run `hronir --help` to see all commands.

---

## 6 Voting-token maths

- Every **path UUID** is a one-shot voting token.
- **Voting power** = `ceil(sqrt(position))` (max one vote per position).
- Token burns after first use; unused power is lost.
- Position 0 tokens don’t exist.

---

## 7 Diary ↔ Quijote interlock 🧩

### 7.1 Reference edition

- **File**: `data/quijote_ref.txt`
- **SHA-256**: `9f03d754…` (stored as `REF_QUX_SHA`)
- UTF-8, stripped—no extra whitespace.

### 7.2 Position mapping

| Canon pos. | Source         | Rule                            |
| ---------- | -------------- | ------------------------------- |
| 0          | Manifesto stub | immutable                       |
| 1, 3, 5…   | Diary fragment | user-provided hrönir            |
| 2, 4, 6…   | Quijote slice  | next 2 000 chars from reference |

### 7.3 Engine hook

After each cascade: if the next canonical position is even and empty,
`interlock.py` copies the next slice, flags it `is_ref=True`, and appends it.

---

## 8 LLM prompt template

```
SYSTEM: Produce deterministic output only.
USER:
Below is the canonical diary of Pierre Menard (odd slots) interleaved
with locked excerpts from Don Quijote (even slots). Using ONLY this
text, reproduce Miguel de Cervantes’ novel verbatim, no commentary.

=== CANON ===
{full_canon_text}
=== END ===
```

- Parameters: `temperature=0.0`, `top_p=1.0`, `seed=42` (if supported).
- `cli.test-quixote` concatenates canon, calls the model, stores `llm_sha`; success when `llm_sha == REF_QUX_SHA`.

---

## 9 Running with an open-weight model

### Local (GPU/CPU)

```bash
export MODEL_ID=meta-llama/Meta-Llama-3-8B-Instruct
python -m vllm.entrypoints.openai.api_server --model $MODEL_ID --seed 42
export MODEL_ENDPOINT=http://127.0.0.1:8000/v1
```

### Hugging Face Inference API

```bash
export HF_TOKEN=…
export MODEL_ENDPOINT=https://api-inference.huggingface.co/models/phi-3-mini-128k-instruct
```

`menard_ext/llm_backend.py` reads `MODEL_ENDPOINT`.

---

## 10 Testing

```bash
pytest -q
```

Key checks:

- pos 0 voting guard
- duel flow increments OpenSkill ratings
- fixture canon → correct Quijote SHA

---

## 11 Contributing

1. Diary hrönirs ≤ 4 000 tokens.
2. Keep continuity; random nonsense loses duels.
3. One PR per logical change; CI (Ruff + Black + pytest) must pass.

See `CONTRIBUTING.md` for details.

---

## 12 License

- Code: MIT
- Diary content: CC-BY-SA 4.0 (unless PD)
- Reference Quijote: Project Gutenberg (public domain)
- LLM weights: respect each model’s open license (Llama 3, Mistral-AI OSL-v0.1, etc.).

_Happy dueling—may your Menard fragment survive to unlock the one true Quijote!_
