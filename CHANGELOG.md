# Changelog

All notable changes to the scenario packages published from this repository are documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
Versions follow the cadence release scheme `YYYY.MM.PATCH`, where `YYYY.MM` is the SAS Viya cadence
the package targets.

## [Unreleased]

## [2026.06.1] — not yet released

Targets SAS Viya cadence **2026.06**.

### Added

- Initial scenario package, extracted from the `sas-validation-scenarios` repository so that scenario
  content is versioned and released independently of the execution framework.
- Package structure: root `release-manifest.yaml`, `scenarios/`, `workload-definitions/`, `resources/`.
- Package manifest fields (`id`, `display-name`, `runtime`, `entrypoints`, `workload-definitions`,
  `required-resources`) added to each scenario's existing `scenario-config.yaml`.
- Five scenarios: `helloviya-simple`, `helloviya-advanced`, `sas-studio`, `sas-visual-analytics`,
  `sas-viya-cli`.
- `.gitignore` covering macOS metadata, Python bytecode, and locally built release artifacts.

### Changed

- `sas-viya-cli` now declares its scenario-local `resources/test.sas` in `required-resources`, matching
  the file its `type: resource` dependency actually resolves to.

### Removed

- `technology-scenarios/compute` — carried no tests and was not part of the release.
- Solution scenarios, including SAS Insurance Capital Management, together with their resources and
  workload definitions. These move to a separate scenario package repository.
- Duplicate `resources/sas_program/test.sas` at the package root, superseded by the scenario-local copy.

### Known issues

- `sas-viya-cli` workload generation fails resolving its `cli-profile` dependency; the declared path does
  not match the framework's template location. The other four scenarios generate successfully.
