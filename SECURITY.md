<!--
SPDX-FileCopyrightText: Copyright (c) 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
SPDX-License-Identifier: CC-BY-4.0 AND Apache-2.0
-->

# Security Policy

## Reporting a Vulnerability

NVIDIA is committed to addressing security issues in this project responsibly. If you believe you have found a security vulnerability in **Cosmos Data Factory** or in any skill published from this repository, please report it to the NVIDIA Product Security Incident Response Team (PSIRT) rather than opening a public issue or merge request.

- **Email:** [psirt@nvidia.com](mailto:psirt@nvidia.com)
- **Web:** [https://www.nvidia.com/en-us/security/](https://www.nvidia.com/en-us/security/)

When reporting, include enough detail for the team to reproduce and triage the issue:

- A description of the issue and its potential impact.
- The skill, script, or file affected (path within this repo).
- Steps to reproduce, expected behavior, and observed behavior.
- Any proof-of-concept, logs, or screenshots that help diagnose the issue.
- Your contact information for follow-up.

PSIRT will acknowledge receipt and coordinate disclosure with the maintainers of this project. Please do **not** disclose the issue publicly until NVIDIA has had a reasonable opportunity to investigate and remediate.

## Scope

This policy covers:

- The deprecated OSMO-orchestrated skills preserved at tag `paidf-26.10`.
- Documentation in this repository where security-relevant guidance is provided.

It does not cover third-party software, container images, or services referenced by these skills. Vulnerabilities in those should be reported to their respective maintainers; please still notify PSIRT if the issue is exploitable through this project's skills.

## Supported Versions

Skills in this repository are versioned via the `metadata.version` field in each `SKILL.md` frontmatter. The current `main` branch is the only branch that receives security fixes. Older tagged releases are not patched in place; consumers should pull the latest published skill from the catalog.

## Getting Help

For non-security questions about contributing or using this project, see [CONTRIBUTING.md](CONTRIBUTING.md) and the per-skill `references/` directories.
