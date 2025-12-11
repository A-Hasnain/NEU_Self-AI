# Week 5 – PAG-style RL with A*-PO on MATH

This repository contains my Week 5 assignment implementation for training Qwen2.5‑1.5B‑Instruct on the MATH dataset using an A*-PO–style reinforcement learning loop and evaluating on MATH‑500.

## Overview

- **Model:** `Qwen/Qwen2.5-1.5B-Instruct` as the policy and as a frozen reference model for value estimation.
- **Datasets:**
  - Training: `EleutherAI/hendrycks_math` (algebra split as a proxy for the full MATH dataset).
  - Evaluation: `HuggingFaceH4/MATH-500`.
- **Algorithm:** A simplified A*-PO setup:
  - Offline: estimate \(V^*(x)\) by sampling a few rollouts per question from the frozen reference model and taking the max reward.
  - Online: for each question, sample one trajectory from the current policy, compute reward, compute advantage \(A = r - V^*(x)\), and update the policy with an advantage-weighted log-prob loss (no external RL libraries).
- **Reward:** Binary reward = 1.0 if the gold final answer string appears in the model’s prediction, 0.0 otherwise (simple proxy for MATH correctness).

The current implementation uses small subsets and short training for practicality; matching the full paper’s performance is not the goal for this week.

## Files

- `week5.py`: Main script with:
  - Data loading
  - Offline value estimation
  - A*-PO training loop
  - Simple evaluation (accuracy before/after training) on a small MATH‑500 subset.

## How to run (Colab-friendly)

1. Make sure you have a GPU (e.g., Colab GPU runtime).
2. Install dependencies:
