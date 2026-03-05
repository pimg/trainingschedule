# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Single-file browser app: `index.html` — no build step, no dependencies, no server required. Open directly in a browser to run.

## Architecture

Everything lives in `index.html` in three sections:

1. **`<style>`** — all CSS, including `@media print` rules that hide the input form and produce clean printable tables for PDF export (via `window.print()`).

2. **HTML body** — two top-level divs:
   - `#input-section` — athlete parameters, training template builder, schedule settings, weekly assignment grid
   - `#output-section` — rendered schedule tables (hidden until "Generate" is clicked)

3. **`<script>`** — all application logic, structured as:
   - **State** — single `state` object with `athleteParams`, `scheduleConfig`, `templates[]`
   - **Utilities** — `uid()`, `esc()`, `fmtDur()`, `toSec()`, `fromSec()`
   - **Calculations** — `paramsForWeek()`, `calcPower()`, `calcHR()`, `zoneLabel()`, `applyProgression()`, `expandToRows()`
   - **State mutators** — direct mutations on `state`, followed by targeted re-renders
   - **Renderers** — `renderTemplateBuilder()`, `renderAssignmentGrid()`, `renderSchedule()`
   - **Init** — `initDefaults()` pre-loads a demo schedule; `populateForms()` syncs state → DOM on load

## Key Concepts

**Phase types** (stored in `template.phases[]`):
- `steady` — fixed duration + `intensity: IntensityRef`
- `ramp` — linear power progression, `startIntensity` + `endIntensity`
- `interval` — N reps of work+rest, expanded into individual rows by `expandToRows()`

**IntensityRef** — `{ refPoint: 'vt1'|'vt2'|'max'|'custom', percent: Number, customWatts: Number }`. Resolved to watts by `calcPower()`.

**HR estimation** — piecewise linear interpolation between (restHR, 0W), (vt1HR, vt1Power), (vt2HR, vt2Power), (maxHR, maxPower). Implemented in `calcHR()`.

**Progressive overload** — `applyProgression(template, weekIdx)` returns a modified template copy for a given week, scaling reps, work/rest durations, and power multiplier according to `template.progression`. Threshold powers (VT1, VT2) also scale weekly via `paramsForWeek(weekIdx)`.

**Re-rendering strategy** — structural changes (add/remove template or phase, change phase type) call `renderTemplateBuilder()` which does a full innerHTML replace of `#template-builder`. Value changes (number/text inputs) update `state` in-place via `oninput` handlers without re-rendering, to preserve focus. The assignment grid similarly re-renders only on structural changes.

**Weekly assignment** — `state.scheduleConfig.weeklyAssignments` is a 2D array `[weekIdx][sessionIdx] → templateId | null`. `syncAssignments()` reconciles it whenever templates or schedule dimensions change.
