---
name: karpathy-guidelines
description: >
  Behavioral guidelines to reduce common LLM coding mistakes. Use when writing,
  reviewing, or refactoring any Rival Client code (Rust or TypeScript) to avoid
  overcomplication, make surgical changes, surface assumptions, and define
  verifiable success criteria. Adapted from Andrej Karpathy's observations on
  LLM coding pitfalls.
license: MIT — see github.com/multica-ai/andrej-karpathy-skills
---

# Karpathy Guidelines — Rival Client

Behavioral guidelines to reduce common LLM coding mistakes, adapted from
[Andrej Karpathy's observations](https://x.com/karpathy/status/2015883857489522876)
on LLM coding pitfalls, tuned for this Rust + React codebase.

**Tradeoff:** These guidelines bias toward caution over speed.
For trivial, self-contained tasks, use judgment and skip steps that don't apply.

---

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing anything:
- State your assumptions explicitly. If uncertain, **ask**.
- If multiple interpretations exist, present them — don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

> **Rival context:** The architecture has strict constraints (< 30 MB idle RAM,
> < 0.4s startup, async-only Rust). Always verify your approach doesn't violate
> these before writing a single line.

---

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: *"Would a senior Rust engineer say this is idiomatic and minimal?"*
If not, trim it.

> **Rival context:** Every extra dependency is RAM. Every abstraction layer is
> startup time. Keep Rust modules focused. Keep React components single-purpose.

---

## 3. Make Surgical Changes

**Change only what needs to change. Don't rewrite what works.**

- Read and understand existing code before touching it.
- Make the smallest diff that solves the problem.
- Don't refactor unrelated code as a side effect of your task.
- Preserve existing comments and docstrings unless they are directly wrong.
- After editing, re-read the surrounding code to confirm nothing broke.

> **Rival context:** The codebase has performance-sensitive paths (startup sequence,
> JVM spawn). Touching unrelated code in those paths can break timing guarantees.

---

## 4. Verify, Don't Assume It Works

**Code that compiles is not code that works.**

After writing any non-trivial change:
1. Run `cargo check` / `pnpm tsc --noEmit` to confirm it compiles.
2. State what observable behavior would confirm it works.
3. If you can't verify it directly, say so explicitly.
4. Don't claim something "should work" — describe what you'd test.

---

## 5. Don't Hallucinate APIs

**Check before using.**

- Never invent Tauri v2 IPC API shapes from memory — verify against docs or existing code.
- Never invent Rust crate APIs — check `Cargo.toml` for the actual version and its docs.
- Never invent Zustand store shapes — read `src/store/` before accessing state.
- When in doubt, read the existing usage in the codebase first.

> Tauri v2 has **breaking changes** from v1. `@tauri-apps/api` import paths changed.
> Always verify with: `grep -r "invoke" src/` to see how IPC is called currently.

---

## 6. State Your Success Criteria

**Before writing code, define what "done" looks like.**

For every task, write one sentence:
> "This is done when [observable behavior] happens."

Examples:
- *"Done when clicking PLAY launches the JVM and the launcher minimizes."*
- *"Done when `cargo clippy` passes with zero warnings."*
- *"Done when the splash window appears in < 100ms on a cold start."*

This prevents scope creep and lets you stop confidently.

---

## 7. One Thing at a Time

**Don't batch unrelated changes.**

- If fixing a bug, fix only that bug.
- If implementing a feature, don't polish unrelated UI.
- Commit each logical unit separately (Conventional Commits: `feat:`, `fix:`, `perf:`).

---

## Quick Reference Checklist

Before submitting any code change, verify:
- [ ] Assumptions are stated (or clarified with the user)
- [ ] The diff is minimal — no speculative extras
- [ ] Existing working code is untouched
- [ ] It compiles (`cargo check` / `tsc --noEmit`)
- [ ] Success criteria is defined
- [ ] No invented APIs — all usages verified against actual codebase or docs
