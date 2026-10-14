# Agent team

Mona's Project Pulse dashboard is coordinated from the GitHub Copilot CLI in a Codespace by a four-agent custom team.

## Orchestrator

**Model:** Claude Opus 4.7 (copilot)  
**Definition:** `.github/agents/orchestrator.agent.md`

Breaks dashboard work into dependency-aware phases, delegates explicit file scopes to specialists, coordinates parallel and sequential work, and verifies the integrated result.

## Planner

**Model:** Claude Opus 4.7 (copilot)  
**Definition:** `.github/agents/planner.agent.md`

Researches the repository and relevant documentation, identifies requirements, dependencies, risks, and edge cases, then produces an actionable implementation plan.

## Designer

**Model:** Gemini 3.1 Pro (copilot)  
**Definition:** `.github/agents/designer.agent.md`

Defines the dashboard's UI/UX, accessibility, information hierarchy, responsive behavior, and polished Project Pulse visual treatment.

## Coder

**Model:** GPT-5.5 (copilot)  
**Definition:** `.github/agents/coder.agent.md`

Implements the assigned application logic and runnable-app support, with explicit errors, deterministic behavior, and validation of completed changes.

The Orchestrator uses the Planner, Designer, and Coder from GitHub Copilot CLI in the Codespace, assigning non-overlapping work in parallel where possible and sequencing dependent work.
