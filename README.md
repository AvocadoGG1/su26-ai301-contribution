# Contribution [1]: [Feature Request: Auto export to html/ipynb](https://github.com/marimo-team/marimo-lsp/issues/181)

**Contribution Number:** [1]  
**Student:** [Andrew Garcia Leopold](https://github.com/AvocadoGG1)  
**Issue:** [GitHub issue link](https://github.com/marimo-team/marimo-lsp/issues/181)
**Status:** [Phase I] [Complete]
**Fork:** [https://github.com/AvocadoGG1/su26-ai301-contribution](https://github.com/AvocadoGG1/su26-ai301-contribution)

---

## Why I Chose This Issue

I chose marimo-lsp issue #181, “Feature Request: Auto export to html/ipynb,” because it connects well with my experience in Python, JavaScript, Git, and web application development. The issue interests me because it involves improving a real developer tool: the marimo VS Code extension. I like that the problem is specific and understandable: users want an easier way to export marimo notebooks to HTML or ipynb directly from the extension workflow.

This issue also matches my learning goals because I want to better understand how VS Code extensions work, how they connect to Python-based tools, and how commands are added to an existing open-source project. From reading the issue and related code, I understand that marimo-lsp already has some notebook export functionality in the codebase, so this contribution would likely involve adding or improving an extension command rather than building the entire export system from scratch. I hope to learn more about TypeScript, editor tooling, command registration, and how open-source projects structure features that connect frontend editor actions with backend notebook functionality.

---
## Understanding the Issue

### Problem Description

The VS Code marimo extension does not currently support marimo’s “auto export outputs” feature. In the regular marimo browser UI, users can enable automatic exporting of notebook outputs to HTML and/or IPYNB. The VS Code extension has some export support already, but it is manual, not automatic.

### Expected Behavior

A user working in a marimo notebook inside VS Code should be able to enable auto export for HTML and/or IPYNB. After running or updating notebook outputs, the extension should automatically save exported snapshots, likely into a `__marimo__` folder next to the notebook file, similar to marimo’s existing behavior.

Example expected output:

my_notebook.py
__marimo__/
  my_notebook.html
  my_notebook.ipynb

### Current Behavior

The extension currently supports manual static HTML export through the `marimo: Export static HTML` command. The LSP backend also already has APIs for exporting HTML and IPYNB from the live notebook session.

However, there is no user-facing setting, command, or background process that automatically exports HTML/IPYNB snapshots while editing or running a notebook. No `__marimo__` folder is created automatically.

### Affected Components

Likely involved files/components:

- `extension/package.json`
  - Declares VS Code commands and configuration settings.
- `extension/src/commands/exportNotebookAsHtml.ts`
  - Existing manual HTML export command.
- `extension/src/commands/publishMarimoNotebookGist.ts`
  - Uses IPYNB export internally for Gist publishing.
- `extension/src/features/RegisterCommands.ts`
  - Registers VS Code commands.
- `extension/src/platform/VsCode.ts`
  - Contains wrappers for VS Code workspace, notebook, filesystem, and event APIs.
- `src/marimo_lsp/api.py`
  - Backend LSP API already supports `export-as-html` and `export-as-ipynb`.

---

## Reproduction Process

### Environment Setup

I set up the local `marimo-lsp` development environment on Windows. The repo requires `uv`, `pnpm`, `just`, Git, and a sibling checkout of `marimo`.

Challenges I ran into:

- Git was not initially available on the shell PATH.
- Git line ending settings needed adjustment to avoid noisy file changes.
- The sibling `marimo` checkout needed to be pinned to the version in `.marimo-version`.
- Frontend dependencies from the sibling `marimo` repo needed to be installed.
- Generated model metadata for `@marimo-team/llm-info` was missing and had to be generated.
- The VS Code extension build needed to complete successfully before using F5.

After setup, I verified:

- `just build` passes.
- `just lint` passes.
- TypeScript tests pass.
- F5 launches the Extension Development Host.

### Steps to Reproduce

1. Open the `marimo-lsp` repo in VS Code.
2. Press `F5` to launch the Extension Development Host.
3. In the Extension Development Host, open or create a marimo notebook.
4. Run a cell that produces visible output, for example: `import marimo as mo; mo.md("# Hello auto export")`
5. Open the Command Palette and search for: `marimo export`, `auto export`, `ipynb`, and `html`.
6. Check the notebook directory for a generated `__marimo__` folder.
7. Observed result: manual `marimo: Export static HTML` exists, but no auto export setting or command exists. No automatic HTML/IPYNB export happens, and no `__marimo__` folder is created automatically.

### Reproduction Evidence

- **Commit showing reproduction:** Reproduction was done locally on branch `fix-issue-181`.
- **Screenshots/logs:** The issue screenshot shows the marimo browser UI “Exporting outputs” setting with HTML/IPYNB checkboxes. I could not find an equivalent setting or command in the VS Code extension.
- **My findings:** The feature is partially supported internally but not exposed as automatic behavior. The backend already has `export-as-html` and `export-as-ipynb`, and the extension already has manual HTML export. The missing piece is a VS Code-side auto export service/configuration that watches notebook output changes and writes HTML/IPYNB snapshots automatically.
---

## Solution Approach

### Analysis

[Your analysis of the root cause - what's causing the issue?]

### Proposed Solution

[High-level description of your fix approach]

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** The VS Code marimo extension already supports manual static HTML export and has backend APIs for exporting both HTML and IPYNB, but it does not support marimo’s auto-export outputs feature. The goal is to add a VS Code-side feature that can automatically write HTML and/or IPYNB snapshots to a `__marimo__` folder next to the notebook after notebook outputs change.

**Match:** Similar existing patterns in the codebase include:

- `extension/src/commands/exportNotebookAsHtml.ts` shows how the extension calls the LSP `export-as-html` API and writes an HTML file.
- `extension/src/commands/publishMarimoNotebookGist.ts` shows how the extension calls the LSP `export-as-ipynb` API.
- `src/marimo_lsp/api.py` already implements backend export methods: `export-as-html` and `export-as-ipynb`.
- `extension/src/features/RegisterCommands.ts` shows how extension features and commands are registered.
- `extension/src/platform/VsCode.ts` provides wrappers for VS Code APIs like notebook events and filesystem writes.
- Existing config toggle patterns for auto-run and auto-reload may be useful examples for adding user-facing configuration and context-aware behavior.

**Plan:**

1. Update `extension/package.json` to add VS Code configuration settings for auto export, such as `marimo.export.autoHtml`, `marimo.export.autoIpynb`, and possibly `marimo.export.autoExportDelayMs`.
2. Add a new extension feature file, likely under `extension/src/features/`, such as `AutoExport.ts`.
3. In the new auto-export feature, subscribe to notebook/document change events and detect when a marimo notebook has updated outputs.
4. Implement debounced auto-export behavior so the extension does not export on every tiny change.
5. Only run auto export for notebooks of type `marimo-notebook`.
6. Reuse the existing LSP export APIs by calling `marimo.api` with `export-as-html` and `export-as-ipynb`.
7. Write exported files into a `__marimo__` folder next to the notebook, for example `__marimo__/notebook.html` and `__marimo__/notebook.ipynb`.
8. Add error handling and logging using Effect logging primitives.
9. Register the new feature in the extension’s main layer setup so it starts when the extension activates.
10. Add tests for path/output filename logic, config behavior, and debounced export triggering.

**Implement:** Working branch:

https://github.com/AvocadoGG1/marimo-lsp/tree/fix-issue-181

Commits will be added to this branch as implementation progresses.

**Review:** Self-review checklist:

- Uses existing Effect-TS patterns instead of ad hoc async code.
- Reuses existing `export-as-html` and `export-as-ipynb` APIs instead of duplicating backend export logic.
- Adds focused configuration settings in `package.json`.
- Keeps changes scoped to auto-export behavior.
- Avoids committing local setup-only files.
- Adds tests for new behavior.
- Runs formatting, linting, and tests before opening a PR.
- Follows project convention of using `pnpm`, `uv`, and `just`.

**Evaluate:** Verification plan:

1. Run static checks with `just lint`.
2. Run relevant TypeScript tests with `just test-ts`.
3. Build the extension with `just build`.
4. Launch the Extension Development Host with `F5`.
5. Create or open a marimo notebook.
6. Enable auto export settings for HTML and/or IPYNB.
7. Run a cell that produces output.
8. Confirm that a `__marimo__` folder is created next to the notebook.
9. Confirm that the expected files are written: `__marimo__/notebook.html` and `__marimo__/notebook.ipynb`.
10. Modify and rerun a cell, then confirm the exported files update automatically.
11. Disable the settings and confirm no auto-export files are written.
---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
