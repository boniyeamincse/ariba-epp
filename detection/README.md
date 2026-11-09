# Detection Content

Versioned detection content and its tests.

| Directory | Contents |
|---|---|
| `rules/` | Ariba-native versioned rules with metadata and false-positive notes |
| `sigma/` | Sigma rule sources and conversion notes |
| `yara/` | YARA rules for file scanning |
| `mitre/` | MITRE ATT&CK mapping, reviewed against evidence |
| `tests/` | Safe test cases and expected results |

Every rule needs a unique ID, severity, supported platform, required telemetry,
versioned logic, provenance, test cases and expected result. Response
recommendations are separate from detection logic. Do not use live malware in
tests.
