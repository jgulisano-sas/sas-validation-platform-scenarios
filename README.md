# sas-validation-platform-scenarios

Validation scenario packages for SAS Viya 4, consumed by the
[sas-validation-scenarios](https://github.com/sassoftware/sas-validation-scenarios) execution framework.

This repository contains **scenario content only** — tests, workload definitions, and the resources they
depend on. The execution framework, generators, and CLI live in the framework repository. The two are
released independently: this repository publishes versioned scenario packages, and the framework installs
and runs them.

## Package layout

```
release-manifest.yaml        # package identity + list of included scenarios
scenarios/                   # one directory per scenario
workload-definitions/        # Locust/Kubernetes workload specs
resources/                   # shared data, dataflows and SAS programs
```

## Included scenarios

| ID | Path | Tests |
|---|---|---|
| `helloviya-simple` | `scenarios/helloviya-simple` | 2 |
| `helloviya-advanced` | `scenarios/helloviya-advanced` | 3 |
| `sas-studio` | `scenarios/application-scenarios/sas-studio` | 5 |
| `sas-visual-analytics` | `scenarios/application-scenarios/sas-visual-analytics` | 1 |
| `sas-viya-cli` | `scenarios/technology-scenarios/sas-viya-cli` | 1 |

`workload-definitions/1_all-basic-wrkld-def.yaml` and `3_all-basic-wrkld-def.yaml` are aggregate
definitions that span several scenarios; they are not owned by any single scenario.

## Versioning and cadence

SAS Viya changes between cadences, so scenario content is maintained per cadence:

- **Branch** — a mutable maintenance line for one cadence, e.g. `2026.06`
- **Tag** — an immutable release cut from that branch, e.g. `2026.06.1`
- **Release asset** — `sas-validation-platform-scenarios-<version>.tar.gz`, plus a SHA-256 checksum

Consumers install by version, never by branch. Use the package whose `viya-cadence` matches your
SAS Viya deployment.

## Using a scenario package

Run these from a checkout of the framework repository, once a repository alias for this repository
has been added to its `scenario-repositories.yaml`:

```bash
# List available releases
sas-validation --config scenario-repositories.yaml \
  list-versions --repository <alias>

# Download and extract a release
sas-validation --config scenario-repositories.yaml \
  install --repository <alias> --version 2026.06.1

# Generate Locust/Kubernetes workload artifacts from an extracted package
sas-validation --config scenario-repositories.yaml \
  generate <package-dir> --scenario helloviya-simple
```

`validate` checks a package without generating anything, and `prepare` writes generator inputs only.

## Authoring a scenario

Each scenario directory contains a `scenario-config.yaml` holding both the package manifest fields and
the legacy generator fields:

```yaml
scenario-config:
  id: my-scenario                  # must match the id in release-manifest.yaml
  display-name: My Scenario
  runtime: locust
  entrypoints:                     # relative to the scenario directory
    - tests/my_test.py
  workload-definitions:            # relative to the package root
    - workload-definitions/my-scenario-wrkld-def.yaml
  required-resources: []           # relative to the package root

  name: my-scenario
  description: What this scenario exercises
  dependencies:
    - type: common
      path: framework/execution/common
      file: events.py
```

Then register it in `release-manifest.yaml`:

```yaml
scenarios:
  - id: my-scenario
    path: scenarios/my-scenario
```

Notes:

- The file must stay named `scenario-config.yaml` — workload definitions reference it by path.
- Keep the `dependencies` block. It drives config map generation and is read separately from the
  manifest fields.
- `entrypoints` resolve from the scenario directory; `workload-definitions` and `required-resources`
  resolve from the package root.

### Where resources belong

Two locations serve different mechanisms:

| Location | Consumed by | Purpose |
|---|---|---|
| `resources/` at the package root | resource setup / data transfer | content provisioned into SAS Viya |
| `resources/` inside a scenario | `type: resource` dependencies | files mounted into the test pod |

A `type: resource` dependency resolves relative to the **scenario** directory, and generation fails if
the file is missing there.

## Validating changes

Validate every scenario against the framework before opening a pull request:

```bash
for id in helloviya-simple helloviya-advanced sas-studio sas-visual-analytics sas-viya-cli; do
  sas-validation validate <package-dir> --scenario "$id"
done
```

## Known issues

- `sas-viya-cli` workload generation currently fails resolving its `cli-profile` dependency. The
  declared path does not match the framework's template location. Tracked for a follow-up fix.

## License

Apache 2.0.
