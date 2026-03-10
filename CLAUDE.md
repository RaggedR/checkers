# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A checkers game with an AI opponent that learns through TD(lambda) self-play, inspired by Arthur Samuel's 1959 program. Pure client-side JS (single-file, no build step, no server needed). Hosted on GitHub Pages.

## Commands

```bash
# Play the game — just open in a browser
open index.html

# Train AI from CLI (default 200 games + 20-game match vs defaults)
python ai.py

# Train with specific game count
python ai.py train 500
```

Training requires Python with no external dependencies. After training, copy the new weights from `weights.json` into the `WEIGHTS` array in `index.html`.

## Architecture

Single-file frontend + Python training tools:

- **`index.html`** — Complete game: engine, AI, and UI all inline. Game engine ported from `checkers.py` — board is 32-element array (only dark squares), precomputed `ADJACENT`/`JUMPS` tables, DFS multi-jump captures, mandatory capture rule. AI ported from `ai.py` — 10 hand-crafted features, linear evaluation, minimax with alpha-beta pruning at depth 5. Learned weights hardcoded. Supports click-to-move and drag-and-drop.

- **`checkers.py`** — Python game engine (used by training only). Board representation and move generation identical to JS port.

- **`ai.py`** — Training code. `extract_features()` computes 10 features from black's perspective. `train_td()` does two-phase TD(lambda): 40% vs random, 60% self-play. Also contains `play_match()` for evaluation.

- **`weights.json`** — Learned weights output from training. The JS version has these hardcoded.

## Key Design Decisions

- All features are computed from black's perspective. When the learner plays white during training, the value function is negated (`V = learner_color * dot(w, f)`) and the gradient flips sign accordingly.
- Terminal values differ by context: `+-1000` for search (to make wins/losses dominate), `+-1.0` for TD training (to keep updates stable).
- Move ordering sorts by move length descending (multi-jumps first) to improve alpha-beta pruning.
- AI thinking runs via `setTimeout` with a 600ms delay so the UI can show the "thinking" spinner before the synchronous minimax blocks the thread.
- The Flask version is preserved at git tag `flask-version`.
