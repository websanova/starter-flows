# Agent Guide

## Commands

Only set up with Claude Code, through a single `/ex` command in `.claude/commands/`. Writes stay behind an explicit `/ex`, so the agent proposes in chat first and only touches files once told. Kill or modify it as needed for your workflow.

## Guidelines

The `<flow-guidelines>` and `<behavior-guidelines>` blocks in [CLAUDE.md](../CLAUDE.md) are a generic starting point, included as a reference for new projects. The flow block holds the repo rules and the templates for flows, docs and refs. The behavior block covers response style, accuracy and code output, and reflects one way of working. Tweak them as needed, delete them, or move the behavior block to a git ignored `CLAUDE.local.md` or your personal `~/.claude/CLAUDE.md` if it should follow you across projects instead of living in the repo. Nothing regenerates them, so edits stay put.

## Rules

The phase and layering rules live in [flow-structure](../.claude/rules/flow-structure.md), which Claude loads alongside `CLAUDE.md`.
