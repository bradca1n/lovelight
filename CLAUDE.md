Lovelight is an operational workspace for managing window furnishing jobs and the internal workflows that support quoting, coordination, handover, install logistics, and job delivery.

This product is primarily designed for internal teams and operational users, not a consumer audience. The experience should feel clear, efficient, structured, and dependable. It should reduce ambiguity, speed up admin tasks, and help staff confidently move jobs through the workflow.

Claude should behave as a design, product thinking, and implementation assistant for Lovelight. The goal is not just to make screens look good, but to make operational tasks easier, clearer, and harder to get wrong.


## ⛔ WORKFLOW GATE — Read This FIRST (see E-G09, E-G13)

**For ANY non-trivial task** (touches 3+ files, has multiple approaches, involves subagents, or is a security review):

You MUST actually INVOKE (tool call, not text) these two skills BEFORE doing anything else:

1. **`Skill(skill="brainstorming")`** — identifies requirements, attack surfaces, approaches, tool selection. Do NOT substitute your own brainstorming.
2. **`Skill(skill="writing-plans")`** — creates a reviewable plan with phases, tool/MCP/skill assignments, agent structure. Do NOT substitute `EnterPlanMode` alone.

**Checkpoint** — confirm before proceeding:
- [ ] Both skills INVOKED via tool calls (not mentioned in text)
- [ ] Plan lists ALL tools/MCPs/skills, files to read, and subagents with scope
- [ ] Prior work searched (claude-mem if available, or auto-memory/session transcripts)

For trivial tasks (single file, obvious fix, < 3 steps): skip this gate.

## Skills workflow

Skills MUST be invoked via the `Skill` tool — not described in text. Describing a skill without invoking it is error E-G13.
---
| Skill | When to INVOKE (via Skill tool) |
|-------|------------|
| `brainstorming` | **Before any creative work** — new features, components, design tokens, style guide, layout architecture decisions |
| `executing-plans` | **Before any creative work** — Use when you have a written implementation plan to execute in a separate session with review checkpoints|
| `frontend-design` | **Before any creative work** — Use this skill for UI tasks that build web components, pages, or applications.w features, components, design tokens, style guide, layout architecture decisions |
| `implement-design` | **Before any creative work** — Translates Figma designs into production-ready code with 1:1 visual fidelity. Use when implementing UI from Figma files, when user mentions "implement design", "generate code", "implement component", "build Figma design", provides Figma URLs, or asks to build components matching Figma specs. Requires Figma MCP server connection.|
| `verification-before-completion` | **Before claiming done** — run tests, verify output, check all files |
| `canvas-design` | **Before claiming done** — Create beautiful visual art in .png and .pdf documents using design philosophy. You should use this skill when the user asks to create a poster, piece of art, design, or other static piece. Create original visual designs, never copying existing artists' work to avoid copyright violations. |
| `writing-plans` | **Before implementation** — creates a reviewable plan with phases, tool/MCP/skill assignments, agent structure |

**Required workflow chain** for non-trivial tasks (each step = actual `Skill` tool call):
```
Skill("brainstorming") → Skill("writing-plans") → Skill("frontend-design") →
  Skill("implement-design") → Skill("verification-before-completion")
```

## ⛔ Figma Design Rules — Mandatory Checks

**NEVER present Figma design work as complete without passing ALL of these checks.**

### Pre-build: Resolve before creating nodes
1. **Fetch variables first** — call `figma_get_variables` to get Semantic + Fixed collection variable IDs. Build a lookup map before creating any nodes.
2. **Check available fonts** — scan existing text nodes for `fontName`. Only use confirmed font family + style combinations. Available: Inter Regular, Inter Medium, Serrif Condensed Light. Never assume other styles exist.
3. **Check available text styles** — load via `figma.getLocalTextStylesAsync()`. All text must use styles named `Text-{weight}/{size}` (e.g. `Text-semibold/2xl`, `Text-normal/xs`).
4. **Search for components** — call `figma_search_components` before building any UI element. Use component instances for buttons, inputs, selects, toggles, icons — never raw frames.

### During build: Bind everything to tokens
- **Colors** → `figma.variables.setBoundVariableForPaint()` — never raw hex values
- **Spacing** → `setBoundVariable('itemSpacing', spaceVar)` — never empty spacer frames
- **Radius** → `setBoundVariable('topLeftRadius', radiusVar)` etc. — never raw px values
- **Text** → `node.setTextStyleIdAsync(styleId)` — never raw fontSize/fontName/lineHeight
- Structure every `figma_execute` as: resolve IDs → create nodes → bind in same pass

### Post-build: Verify before claiming done
Run these scans via `figma_execute` BEFORE every completion claim:
1. **Tokens audit** — scan all nodes for hardcoded fills, strokes, spacing, radius. Zero tolerance for unbound values.
2. **Font audit** — scan all text nodes for font family + style. Flag anything not in the approved list.
3. **Text style audit** — verify all text nodes are linked to a Figma text style, not raw properties.
4. **Component audit** — flag raw frames that should be component instances.
5. **Component internals** — after setting instances to FILL, verify child sizing wasn't broken (e.g. chevron icons inside selects should stay FIXED).

### On code-to-Figma import
After any import, run a component audit pass — match elements to existing components, swap instances, and flag new components that need to be created.