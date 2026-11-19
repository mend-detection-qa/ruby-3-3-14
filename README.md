# ruby-3.3.14-bundled-bundler-pin

## Feature exercised

This probe verifies Mend correctly detects dependencies when a project
declares `ruby "~> 3.3.14"` and uses the bundled Bundler version
(2.5.22) that ships with Ruby 3.3.14, with `.whitesource`
`scanSettings.versioning` pinned to match the exact toolchain.

## Expected dependency tree

- `rack` 3.1.8 — direct, default group, source: registry
- `sinatra` 3.2.0 — direct, default group, source: registry
  - `mustermann` 3.0.0 — transitive via sinatra, default group
  - `rack-protection` 3.2.0 — transitive via sinatra, default group
    - `rack` 3.1.8 — shared with direct dep (single resolved copy)
  - `tilt` 2.3.0 — transitive via sinatra, default group
- `rspec` 3.13.0 — direct, test group, source: registry
  - `rspec-core` 3.13.0 — transitive via rspec, test group
    - `rspec-support` 3.13.0
  - `rspec-expectations` 3.13.0 — transitive via rspec, test group
    - `diff-lcs` 1.5.1
    - `rspec-support` 3.13.0
  - `rspec-mocks` 3.13.0 — transitive via rspec, test group
    - `diff-lcs` 1.5.1
    - `rspec-support` 3.13.0

Mend must report:
- All 11 gems with their exact versions
- All gems sourced from `registry` (rubygems.org)
- `rack` and `sinatra` and their transitives in the default group
- `rspec` and all its transitives in the test group
- `RUBY VERSION ruby 3.3.14p0` and `BUNDLED WITH 2.5.22` from the
  lockfile reflected in `project_metadata`

## Mend config

Bucket A — `.whitesource` pins `bundler: "2.5.22"` and
`ruby: "3.3.14"`. No `whitesource.config` ships; `configMode` is
`"AUTO"`.

Both versions are sourced from `Gemfile.lock`:
- `bundler` from `BUNDLED WITH 2.5.22`
- `ruby` from `RUBY VERSION ruby 3.3.14p0` (patchlevel suffix dropped)

`ruby-bundler` has no dynamic version detection from the manifest, so
every probe in this plugin must ship a `.whitesource` with
`scanSettings.versioning` pinning both `bundler` and `ruby` to exact
versions. This prevents `install-tool` from drifting to a different
toolchain patch, which would silently produce a different resolved tree.

## Failure modes to watch

- Mend ignores `scanSettings.versioning` and falls back to its default
  Bundler install-tool version, potentially producing a different
  resolved tree.
- `RUBY VERSION` or `BUNDLED WITH` lockfile sections not propagated
  to `project_metadata`.
- Lockfile format for Bundler 2.5.22 not recognized — Mend truncates
  or mis-parses the file.
- Falls back to manifest-only (pre-step off) parsing, losing transitive
  dependencies.

## Probe metadata

| Field                 | Value                           |
|-----------------------|---------------------------------|
| Pattern               | ruby-3.3.14-bundled-bundler-pin |
| Category              | bundled_pm_change               |
| Status                | generated                       |
| Schema version        | 1.2                             |
| pm_version_under_test | 2.5.22                          |
| Ruby version          | 3.3.14                          |
| Bundler version       | 2.5.22                          |
| Generated             | 2026-11-19                      |
