---
description: Scaffold the 5bot working files into the current project.
---
Initialize the 5bot workflow in the current project (see the `five-bot` skill).

**Workflow overview:** Product Bot defines scope. UX Bot designs experience. Architect Bot plans build. Dev Bot implements. QA Bot reviews. Humans approve at gates. Each role reads shared Markdown state files and produces one artifact per stage.

Create the following seven files in the project root, each with exactly the content shown between the markers. Do NOT overwrite any file that already exists - report which you skipped. Then add the line `@project-state.md` near the top of the project's CLAUDE.md (create CLAUDE.md if absent), set Current Stage to "Product" in project-state.md, and tell me to run `/product`.

===== project-state.md =====
# Project State
## Project Name
## One-Sentence Concept
## Target Users
## Current Stage
## Approved MVP Features
## Explicitly Out of Scope
## Current Tech Stack
## Current Active Ticket
## Major Decisions (Product / UX / Technical)
## Open Questions
## Known Risks
## Last Updated / By

===== decisions.md =====
# Decision Log
<!-- Decisions and human-gate approvals. Newest on top. -->
## Decision / Approval: [Title]
- Date / By / Stage:
- Status: Proposed | Approved | Approved w/ Changes | Revise | Rejected | Replaced | Deferred
- Decision:
- Reason:
- Alternatives considered:
- Consequences:
- Required changes (if any):

===== handoff.md =====
# Handoff
<!-- Transient. Overwrite each stage. -->
## Stage Completed
## Bot
## What Changed
## Artifacts Created / Updated
## Open Questions / Risks
## Human Approval Required?
## Next Bot + Instructions

===== product.md =====
# Product
## Product Name
## One-Sentence Pitch
## Problem
## Target Users
## User Goals
## Value Proposition
## MVP Features
## Out of Scope (Future / Recommended Later)
## Success Criteria
## Risks
## Open Questions

===== ux.md =====
# UX
## Primary User Goals
## Main User Flows
## Screen List
## Navigation Model
## Screen-by-Screen Detail
## Forms & Fields
## Buttons & Actions
## Empty / Error / Success States
## Usability Concerns

===== architecture.md =====
# Architecture
## Tech Stack
## System Architecture
## Data Model
## API Spec
## Auth & Permissions
## Integrations
## Implementation Plan
## Tickets
<!-- Per ticket: Goal - relevant requirements/UX/data - Acceptance Criteria - files likely involved - out of scope -->

===== dev-qa.md =====
# Dev & QA
## Backlog / Tickets
## Developer Notes
<!-- Per ticket: summary of changes - files created/modified - assumptions - known limits - how to run - how to test -->
## Review Notes
<!-- Per review: verdict - acceptance-criteria table - findings -->
## Bug List
<!-- Bug: severity - steps - expected - actual - likely cause - recommended fix - related ticket -->
## Release Checklist
