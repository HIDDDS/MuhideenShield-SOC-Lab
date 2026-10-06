# Detection development status

The lab has demonstrated manual Splunk searches and existing Wazuh authentication and sudo processing. No custom correlation rule or automated response has been validated in this repository.

## Searches used

```spl
index=* "malicious_hacker"
```

This retrieves events containing the controlled test username across accessible indexes. It is an investigation search, not a threshold-based brute-force alert.

## Planned validation

- Define a failed-login threshold and time window, then test both expected and suspicious scenarios.
- Correlate failures and later success using the account, source, target, and normalized timestamps.
- Document false positives, evidence, limitations, and relevant ATT&CK mapping.

[Read the complete project walkthrough](../README.md) · [Splunk investigation](../documentation/05-log-analysis/ssh-failed-login-investigation.md)
