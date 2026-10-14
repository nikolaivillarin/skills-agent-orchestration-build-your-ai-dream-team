# Project Pulse implementation plan

## Summary

Mona's Project Pulse is a lightweight, static contributor dashboard. It will load
structured project data from JSON and present each project in a polished,
responsive card layout so contributors can quickly identify active work, its
owner, status, recent activity, priority or risk, and a short
contributor-friendly summary.

The **Orchestrator** uses this plan to coordinate the **Designer** and
**Coder** after the **Planner** has defined the work. File scopes must remain
exclusive in a phase so agents do not overwrite each other's work. No agent
stages, commits, or pushes changes.

## File ownership and implementation contracts

| File | Owner | Contract |
| --- | --- | --- |
| `docs/project-pulse-plan.md` | Planner | This planning record: requirements, assignments, dependencies, risks, and validation. It is documentation only, not dashboard implementation. |
| `app/project-data.json` | Coder | Valid JSON with one top-level `projects` array. Every project object must provide `name`, `owner`, `status`, `recentActivity`, and `priority`; it should also provide the brief's contributor-friendly summary field (for example, `summary`). Use deterministic representative data with a useful range of statuses and priorities. |
| `app/index.html` | Coder | Static dashboard entry page that identifies Project Pulse, links `styles.css`, fetches `project-data.json`, and renders one project card per valid project. It must visibly render name, owner, status, recent activity, priority, and summary. It owns explicit loading, fetch/parse/error, malformed-data, and empty-project-list states, with a usable message rather than a blank page. |
| `app/styles.css` | Designer | Polished visual treatment for the page and the Coder's stable HTML hooks. It must define `.dashboard` and `.project-card`, plus responsive styling, readable hierarchy, rounded cards/shadows, accessible status badges, and priority treatment that does not depend on color alone. |
| `.vscode/launch.json` | Coder | Strict JSON (no comments or trailing commas) defining **Run Project Pulse Dashboard**. It serves the `app/` directory on deterministic port `5500`, uses `cwd` `${workspaceFolder}/app`, and opens `index.html` rather than showing a directory listing. |

### Shared interface decisions

Before implementation, the Designer supplies the Coder with the visual
information hierarchy and stable class/attribute expectations. The Coder then
keeps those hooks in `app/index.html`; the Designer styles only the assigned
CSS file. The proposed card order is: project name and priority, summary,
owner/status metadata, then recent activity. Status and priority values are
present as text as well as visually differentiated badges.

The Coder should use browser-native HTML, CSS, and JavaScript in
`app/index.html`; no framework or build step is required. Because `fetch()` of
the JSON will not work reliably from `file://`, normal validation and use must
go through the configured local HTTP server.

## Ordered phases

### Phase 1 — confirm requirements and lock interfaces

**Owner:** Orchestrator, using this Planner deliverable.

1. Reconfirm the project brief, required files, data schema, and launch
   requirements.
2. Give the Designer exclusive scope over `app/styles.css` and request a
   written handoff of layout, responsive breakpoints, class hooks, badge
   semantics, contrast, focus, and non-color priority/status cues.
3. Give the Coder exclusive scopes over `app/index.html`,
   `app/project-data.json`, and `.vscode/launch.json`. Include the locked
   Designer handoff before code is written.

**Dependency:** This phase completes before the Coder finalizes markup, because
the HTML must retain the Designer's agreed hooks. The Designer can begin
styling only once those hooks are available in the markup, unless the
Orchestrator asks only for design guidance rather than CSS edits.

### Phase 2 — prepare data and runnable configuration

**Owner:** Coder.

1. Create `app/project-data.json` with a top-level `projects` array and
   complete fields for each sample project:
   `name`, `owner`, `status`, `recentActivity`, `priority`, and `summary`.
2. Create `.vscode/launch.json` as strict JSON for **Run Project Pulse
   Dashboard**, serving `app/` at port `5500`, setting `cwd` to
   `${workspaceFolder}/app`, and targeting `index.html`.
3. Parse the JSON and inspect the launch configuration before handoff.

**Parallel decision:** These two files have non-overlapping scopes and no
runtime dependency on each other, so one Coder assignment may create both, or
separate assignments may run in parallel if each receives exactly one file.

### Phase 3 — build the dashboard structure and rendering states

**Owner:** Coder, `app/index.html` only.

1. Build semantic page structure with a Project Pulse heading, short
   contributor-oriented introduction, a live status/message region, and a
   `.dashboard` container for cards.
2. Fetch `project-data.json` after the document is ready; validate that the
   response is successful and the payload has an array at `projects`.
3. Render a `.project-card` for every usable project and expose each required
   value as readable text. Escape or construct inserted content safely rather
   than treating JSON text as markup.
4. Start with a loading state; replace it with cards after success. Replace it
   with a concise error state for HTTP/network failures, invalid JSON, or
   invalid top-level data; show a distinct empty state for `projects: []`.
   Avoid leaving stale loading content or partial cards behind.

**Dependency:** Phase 2 data must exist before functional rendering can be
tested. Phase 3 must establish the agreed CSS hooks before the CSS is finalized.

### Phase 4 — apply the visual system

**Owner:** Designer, `app/styles.css` only.

1. Style the shell and `.dashboard` as a polished, responsive dashboard grid
   with intentional spacing and a readable max-width.
2. Style `.project-card` as a visible, scannable card using rounded corners,
   `box-shadow`, clear grouping, and a robust text hierarchy.
3. Style status badges and priority indicators with sufficient contrast,
   visible text labels, and a secondary cue such as icon, label, border, or
   weight; do not convey meaning by color alone.
4. Provide comfortable touch targets, keyboard-visible focus styling for any
   interactive elements, sensible small-screen reflow, and respect for
   reduced-motion preferences if animation is introduced.
5. Style loading, empty, and error states so each is noticeable and readable.

**Dependency:** This is sequential after Phase 3 when the Designer is expected
to edit CSS, because it depends on final markup hooks. No one else edits
`app/styles.css` in this phase.

### Phase 5 — integrate and validate

**Owner:** Orchestrator coordinates; Coder corrects only its assigned runtime,
data, and launch files; Designer corrects only CSS findings.

1. Run static checks and start the configured preview.
2. Test normal data and each state by temporarily serving controlled data or
   simulating failed/malformed responses without retaining test-only changes.
3. Inspect desktop and narrow viewport views, keyboard navigation, and
   rendered content.
4. Resolve issues in the owning file only, then repeat the affected checks.

**Dependency:** This phase is sequential after all earlier output is
integrated. Cross-file findings are triaged by owner; do not give two agents
the same file to repair concurrently.

## Designer responsibilities

- Establish a clear scan order for contributor decisions: title/context,
  project identity and priority, concise summary, then owner, status, and
  activity details.
- Deliver and implement polished responsive styling solely in
  `app/styles.css`, including `.dashboard`, `.project-card`, badges, priority
  treatment, state messaging, focus visibility, and mobile layout.
- Specify accessible color contrast and redundant text or visual cues for
  statuses and priorities.
- Report design decisions and validation recommendations to the Orchestrator;
  do not change HTML, JSON data, launch configuration, or this plan.

## Coder responsibilities

- Implement valid representative JSON, deterministic client-side rendering,
  and explicit loading/error/empty states in the assigned application files.
- Keep card markup aligned with the Designer's stable hooks and expose all
  required project information as text.
- Create the strict JSON launch configuration and ensure it serves `app/` at
  port `5500` and opens `index.html`.
- Validate data parsing, runtime rendering, and preview behavior; report
  precise failures and residual risks. Do not edit `app/styles.css` when that
  file belongs to the Designer.

## Dependencies and safe parallelism

- **Sequential:** Planner research and this plan precede orchestration.
  Design interface decisions precede final HTML hooks. Valid data precedes
  end-to-end rendering tests. HTML hooks precede final CSS implementation.
  Integration validation follows all implementation work.
- **Can run in parallel:** Once the Coder has the agreed schema, JSON authoring
  and launch-file authoring have separate file scopes. The Designer may prepare
  a written design specification while the Coder prepares JSON and launch
  configuration. After HTML hooks are locked, CSS work is isolated from
  subsequent JSON or launch refinements.
- **Must not run in parallel:** Two agents must never edit the same file.
  CSS editing and HTML-hook changes are sequential. Runtime integration is
  sequential because it depends on data, markup, CSS, and launch behavior
  together.

## Edge cases and risks

- A missing `project-data.json`, non-OK HTTP response, invalid JSON, missing
  `projects`, or a non-array `projects` value must yield an error state.
- An empty `projects` array is valid but requires a distinct empty state.
- Individual records may be incomplete, have unexpected status/priority
  values, or include long names, owners, summaries, or activity text. Rendering
  should not crash or overflow; show a safe fallback or omit only the damaged
  record with an understandable message.
- JSON values are content, not HTML. Rendering must avoid HTML injection.
- A local `file://` open can make JSON loading fail; preview via the launch
  server, not a direct file open.
- Hard-coded localhost URLs, an occupied port `5500`, unavailable server
  command, or a launch configuration that opens `/` instead of `index.html`
  can prevent a usable preview. Verify the actual command and URL in VS Code.
- Color-only badge semantics, insufficient contrast, hover-only information,
  and dense mobile cards would exclude users; test at small widths and with a
  keyboard.

## Validation expectations

### Static validation

- Confirm all five assigned paths exist and that no out-of-scope files are
  needed for the static app.
- Parse `app/project-data.json` with a JSON parser. Verify one top-level
  `projects` array and that every record includes non-empty `name`, `owner`,
  `status`, `recentActivity`, and `priority` values; verify summaries are
  present for the contributor-friendly context.
- Parse `.vscode/launch.json` as strict JSON. Confirm the configuration name is
  exactly **Run Project Pulse Dashboard**, its `cwd` is
  `${workspaceFolder}/app`, it uses port `5500`, and it opens `index.html`.
- Inspect `app/index.html` for the Project Pulse title, references to
  `styles.css` and `project-data.json`, semantic card/status markup, and
  explicit loading, error, and empty handling. Inspect `app/styles.css` for
  `.dashboard`, `.project-card`, responsive rules, `border-radius`, and
  `box-shadow`.

### Runtime and data validation

- Start the VS Code **Run Project Pulse Dashboard** configuration. Confirm the
  server root is `app/`, the browser opens `index.html`, and the initial view is
  the dashboard rather than a directory listing.
- At `http://localhost:5500/` (or the exact launch URL), ensure the fetch
  succeeds, the loading message is replaced, and card count equals the JSON
  project count.
- Confirm every displayed card includes its name, owner, status, recent
  activity, priority, and contributor summary.
- Exercise or simulate loading, network/HTTP failure, invalid JSON,
  malformed top-level payload, empty array, and an incomplete/long-value
  record. Each result must be visible, stable, and understandable.

### Visual, responsive, and accessibility validation

- Review a desktop viewport for a clearly recognizable Project Pulse dashboard,
  consistent cards, readable spacing, visible status badges, and unambiguous
  priority treatment.
- Review a narrow mobile viewport and an intermediate width. Cards must reflow
  without horizontal scrolling, clipped text, or overlapping badges.
- Check text/background and badge contrast, especially for every status and
  priority variation. Ensure labels communicate their meaning without color.
- Navigate with a keyboard; focus must remain visible and logical for all
  interactive elements. Verify page structure uses a meaningful heading and
  live loading/error messaging is announced appropriately where supported.
- If animation is used, verify reduced-motion behavior; otherwise, avoid
  unnecessary motion.

## Open questions for the Orchestrator

The brief does not prescribe the exact allowed values for `status` or
`priority`, the exact data count, or a server/debugger type for the VS Code
launch configuration. Use a small, representative, deterministic sample and
a locally available static server command. If a launch type or command is not
available in the learner's Codespace, report the blocker and select the
simplest documented compatible alternative while preserving the required
configuration name, `cwd`, port, and `index.html` behavior.
