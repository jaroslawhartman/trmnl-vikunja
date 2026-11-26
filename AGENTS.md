# Development notes for agents

## Project and privacy

This repository is a TRMNL private plugin for Vikunja. Preserve user changes and keep the project usable without a serverless transform. `.env` is local and ignored; it can contain a real server and API token. Never print its values, commit it, put credentials into an import ZIP, or publish a real server URL. Use `https://vikunja.example.com` and fictional IDs in documentation and examples. The image in `img/` is the user's sample screenshot; preserve it unless asked to change it.

## Architecture

- `src/settings.yml` defines the form fields and dynamic polling URLs. Each configured `project:view` pair produces three requests, in this exact order: project metadata, open tasks for the selected view, and tasks completed today from the project endpoint. TRMNL polls URLs in parallel and exposes responses as top-level `IDX_0`, `IDX_1`, etc.; do not assume a `merge_variables` object is available in the Liquid view.
- The four viewport files explicitly pass the corresponding `IDX_*` responses to the `vikunja_view` template in `src/shared.liquid`. Keep the three-request indexing and all four files synchronized when changing polling.
- Open tasks use `/api/v1/projects/{project}/views/{view}/tasks` with ascending `position`. Vikunja positions are per view; the global task endpoint does not reliably provide this ordering. Project and view IDs differ; the Projects field accepts comma-separated ID pairs, not names.
- The selected view can independently exclude completed tasks, even when a request filter asks for them. A separate `/api/v1/projects/{project}/tasks` request supplies completed-today tasks; the switch controls display, not fetching. The advanced filter applies to both requests, and date filtering uses TRMNL's `time_zone_iana` (UTC fallback).
- Task hierarchy comes from `related_tasks.parenttask`. Shared markup reconstructs two subtask levels from returned tasks. Filtered-out parents result in visible children becoming roots; deeper descendants are omitted. There are at most six configured projects and one page of up to 100 tasks per request.
- Full view uses CSS multi-column layout with `column-fill: auto` to preserve project/task sequence and fill the left column before the right. An earlier TRMNL overflow/group-header implementation interleaved project headers and tasks in practice; do not reintroduce it without testing multiple projects and a column break. Smaller views use one column. Excess content is clipped. Keep the bottom title bar a sibling of the layout.

## Validation and releases

Validate changes against both empty and populated API responses, multiple projects, nesting, view ordering, and the Done today toggle. Check the full and three mashup layouts in the TRMNL editor or a compatible `trmnlp` preview; archive integrity or YAML parsing alone does not prove the screen renders correctly. The latest layout changes were not locally render-tested because the TRMNL preview runtime was unavailable.

For live API checks, the local `.env` provides `VIKUNJA_SERVER` and `VIKUNJA_TOKEN`. Use them to validate requests when the task authorizes access to the instance, without echoing credentials or task contents. A prior read-only test confirmed that the general project endpoint could return a task completed today while the selected list view excluded it. This is an observation from one instance, not a guarantee for every version.

Users download the import ZIP from the [releases page](https://github.com/jaroslawhartman/trmnl-vikunja/releases). Do not claim visual validation unless it was actually performed.
