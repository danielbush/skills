# TODO — Architecture Skills

## Next Session: Concept Extraction from Nullables Algorithm

### Goal
Walk through the nullables algorithm (SKILL.md lines 75-158) and extract named concepts (entities) using UPPER_SNAKE_CASE convention (e.g., DUAL_FACTORY, DELAYED_INSTANTIATION, FACTORY_OBJECT). Let the vocabulary unfurl naturally from the algorithm rather than defining it top-down.

### Steps
1. **Read the algorithm** — Go through `skills/nullables/SKILL.md` lines 75-158 step by step
2. **Identify concepts** — Each time a distinct pattern/technique appears, name it as an UPPER_SNAKE_CASE entity
3. **Build a concept index** — Brief definition for each, with a pointer back to where it appears in the algorithm
4. **Identify actionable sequences** — Which groups of concepts form a workflow that could become a skill command? (e.g., WRAP_INFRASTRUCTURE might chain: identify I/O → create INFRASTRUCTURE_WRAPPER → add EMBEDDED_STUB → add CONFIGURABLE_RESPONSE → add OUTPUT_TRACKER)
5. **Draft one or more skill commands** — Based on the workflows identified above
6. **Update PRD.md** — Record decisions and any new open questions

### Notes
- Let concepts emerge from the algorithm — don't force a taxonomy
- UPPER_SNAKE_CASE for all entity/vocab names (e.g., CASCADING_NULLABLES, LOGIC_SANDWICH)
- Focus on algorithmic approaches first; vocabulary follows from that
- See PRD.md for broader context (two-layer approach, skill-creator 2.0 patterns)
