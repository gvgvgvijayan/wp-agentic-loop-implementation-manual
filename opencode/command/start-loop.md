---
description: Start the agentic loop. If a spec path is given, produce a plan and stop for approval. If no spec is given, run the requirement-asker first.
agent: wp
---

If $1 is provided, read the spec at $1, produce a plan, and stop for approval. If $1 is
empty, trigger the requirement-asker to interview the user and generate the spec first.
Do not edit code until the plan is approved.
