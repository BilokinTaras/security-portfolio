# Security portfolio · Taras Bilokin

Security researcher and penetration tester with a QA and DevOps background. This page collects the security work I
did in 2026, how I approach an engagement, and the tools I use. Client names and technical details are withheld
under NDA; [the note at the end](#a-note-on-confidentiality) explains what is left out and why.

## At a glance

| | |
|---|---|
| Engagements in 2026 | **11**: 8 commissioned engagements as part of the H-X Technologies team bug bounty, 3 public bug bounty programs |
| Documented findings | **About 80**, including **1 Critical** and **10 High** |
| Accepted and paid | A High-severity denial of service in the **Tor** core daemon |
| Kinds of work | Source-code audit · web and API testing · Android · external infrastructure · cloud (AWS, GitHub) · anonymity audit · smart-contract review |
| Certification | AWS Certified Solutions Architect – Associate (2026) |

## What I do

**Source-code security audit.** C/C++, Java and Python. Fuzzing (libFuzzer, AFL++) with sanitizer builds
(ASan, UBSan, MSan), static analysis (CodeQL, Semgrep, cppcheck, clang-tidy, Infer) and constant-time testing, then
manual review of the paths the tools cannot reason about.

**Web application and API testing.** Authentication, sessions, SSO/OIDC, access control (IDOR/BOLA, function-level
authorization), injection, XSS, CSRF, SSRF, open redirects and file uploads, in black-, grey- and white-box setups.

**External infrastructure.** The Internet-facing surface of a company: TLS, SSH, mail, DNS, identity providers,
monitoring and management interfaces. Every version-based CVE gets its precondition tested before it is reported.

**Cloud and CI/CD.** AWS account and IAM configuration (Prowler, ScoutSuite, Cloudsplaining, PMapper) and GitHub
organizations and pipelines (secrets, workflow permissions, branch protection).

**Anonymity and OS hardening.** How a privacy system fails open: firewall fail modes, clearnet callers, IPv6,
time sources, control interfaces, build and update integrity.

**Smart contracts and cryptography.** Solidity with Foundry (unit, fuzz, invariant), Slither, Halmos; on-chain
authority review (proxy admins, owners, multisig thresholds); review of MPC and zero-knowledge protocol code.

**Risk and reporting.** CVSS 3.1 vectors with a written justification, adversary models, executive summaries and
full audit reports that also show what was checked and found sound.

## Selected work, 2026

| When | Target | Work | Outcome |
|---|---|---|---|
| Jun – Jul | **Tor** core daemon (C) | Source audit: fuzzing, sanitizers, static analysis, manual review | 1 High (accepted, paid), 2 Low |
| Jun – Aug | Software company | AWS and GitHub security configuration audit | Confidential report |
| Aug | **Coinbase** cb-mpc, HackerOne | Audit of a C++ MPC / zero-knowledge library | Research: 1.16M fuzzing iterations, malicious-peer harnesses, constant-time tests |
| Aug | **Netflix**, HackerOne | Source review of the open-source API gateway, reproducible lab | Research and lab |
| Aug – Nov | **Whonix** | Anonymity audit: 86 repositories, Gateway and Workstation on VirtualBox and KVM | 20 findings (2 High), full audit report |
| Sep | Financial services company | External black-box and Android application | 10 findings (3 High, 1 Medium, 6 Low) |
| Oct | Game developer | External infrastructure and web services | 20 findings (1 High, 3 Medium, 12 Low, 4 Info) |
| Nov | Backup software vendor | Product security assessment on a self-built lab, Windows and Linux | 10 findings (1 Critical, 3 High, 3 Medium, 3 Low) |
| Nov | Energy company | Web application test, full crawl of 2,700+ pages | 9 findings, incl. 3 reflected XSS |
| Nov | **Tether**, bug bounty program | Web application and 16 token deployments across EVM chains | 178 hypotheses tracked, 4 Low reports |
| Nov | Enterprise document-management system with SSO | Grey-box test of a staging environment with role accounts | 3 findings |

## Case notes

### A remotely triggerable denial of service in the Tor core daemon
The audit combined sanitizer builds, fuzzing harnesses and static analysis with a manual read of the protocol code.
The finding that mattered came from the manual part: a memory-exhaustion path that one remote client can drive.
It was proven on an unmodified release with a live proof of concept, framed against the project's recent advisories
to show it was not a duplicate, and **accepted and paid at High severity**. Two Low findings were reported with it.

### Authorization in a backup platform's management API: 1 Critical, 3 High
I built the product in a lab on Windows and Linux and mapped its RPC layer method by method, comparing what each method
was supposed to require with what the server actually enforced. The result was:
- an **unauthenticated takeover of the administrator account** on a fresh install (Critical);
- methods that answer without a session;
- an access check that never runs;
- any user being able to change another account's two-factor settings, the administrator's included.

Every claim was confirmed at runtime, not inferred from code.

### Anonymity audit of Whonix
The question was what could reveal a user's real IP address or reduce their anonymity. I worked from a pinned corpus of
86 repositories and two signature-verified images, and tested runtime behaviour on both hypervisors. The audit filed
**2 High** findings, both paths to the user's real IP, and 18 Low findings. It also documented **124 attack paths
that were ruled out with evidence**, so that the absence of a finding is shown, not assumed.

### External infrastructure: defensible Lows instead of inflated Highs
On a game developer's external surface the High was a mail path with no server authentication, which allows a
man-in-the-middle to read and modify the session. The Mediums were:
- SSH Terrapin on 52 of 61 hosts;
- an identity provider that both reveals admin usernames and has no brute-force protection.

Several components were old enough to fall inside the ranges of serious CVEs. Instead of copying vendor scores I
tested the precondition behind each one, and reported them as patch-management Lows when the severe path was not
reachable.

### Public programs
- **Coinbase cb-mpc.**
  - 1.16 million fuzzing iterations on the public deserializers.
  - Harnesses that run an honest party against a malicious one for two-party and multi-party ECDSA.
  - An empirical soundness check of a zero-knowledge proof, constant-time testing (17/17 pass) and CodeQL.
  - The library held up: no in-scope vulnerability.
- **Netflix.** I reviewed path handling and filter ordering in the open-source API gateway and built a Docker lab that
  reproduces the mechanism with the gateway's own parsing code. The live service is not exposed that way, so it
  ended as research.
- **Tether.** I tested the account application (own test accounts, 2FA, API keys, signed API, password reset) and
  reviewed the authority of 16 token deployments: proxy admins, owners and multisig thresholds. All of it was
  read-only, with no transactions on real networks. I tracked 178 hypotheses to a verdict and submitted 4
  Low-severity reports.

## How I work

Every engagement runs through the same loop:

1. **Scope gate.** The statement of work becomes a written scope. In-scope assets, rules of engagement, rate
   limits, excluded classes and the severity model are all spelled out. Nothing is touched until the scope is
   confirmed.
2. **Recon.** Attack-surface mapping, saved to files and ranked by likely impact.
3. **Hypothesis register.** Every idea gets an entry and a verdict: dead, blocked, primitive or candidate.
   The register is the memory of the hunt, so coverage and gaps are visible at any time.
4. **Validation in two stages.** First, is it real and in scope? Then: prove it, measure the impact, and write the
   finding in the same session.
5. **Reporting.** One document per finding. Each one has a CVSS 3.1 vector with a written justification, reproduction
   steps, evidence and remediation. A lint pass checks precision before anything is sent.
6. **Review and close-out.** An independent triage pass on every finding, then lessons learned.

I run this loop with an **AI-augmented workflow that I designed and maintain myself**. It is built on Claude Code
with Caido and a browser agent wired in, and adds hypothesis tracking, evidence discipline and report rendering on
top. AI does the volume work. Judgment, scoping and verdicts stay with me, and **every finding is validated by hand**
before it is reported. The framework is private.

## Reporting standards

- **Observed versus inferred.** A report says what was seen and labels everything else as inference.
- **Severity you can defend.** CVSS vectors are computed, not estimated, and each metric is justified in a sentence.
  When the demonstrated impact is lower than the formula suggests, the rating goes down and the report says why.
- **Preconditions over banners.** A version string is a lead, not a finding, until the vulnerable path is tested.
- **Negative results count.** Large engagements end with a list of what was checked and found sound.

A public sample of this style is the
[security assessment (PDF)](https://github.com/BilokinTaras/devsecops-quality-gates/blob/main/docs/security-assessment-2026-11-01.pdf)
in [devsecops-quality-gates](https://github.com/BilokinTaras/devsecops-quality-gates). It covers 15 vulnerabilities
reproduced by hand against OWASP crAPI, pinned by regression tests, with CVSS computed per finding, and scanners that
missed all 15.

## Toolbox

| Area | Tools |
|---|---|
| Web and API | Caido, Burp Suite, OWASP ZAP, ffuf, Nuclei, sqlmap, arjun, LinkFinder, jwt_tool, Playwright, SecLists, PayloadsAllTheThings |
| Infrastructure | Nmap, testssl.sh, sslscan, ssh-audit, checkdmarc, subzy, Wireshark |
| Code | CodeQL, Semgrep, cppcheck, clang-tidy, scan-build, CodeChecker, flawfinder, Infer, valgrind, ASan/UBSan/MSan, libFuzzer, AFL++, dudect, jadx |
| Cloud | Prowler, ScoutSuite, Cloudsplaining, PMapper, CloudMapper, TruffleHog, Gitleaks, AWS CLI |
| Web3 | Foundry (forge, cast, anvil), Slither, Mythril, Halmos, solc-select, block explorers and RPC |
| Automation | Python, Bash, TypeScript, Docker, GitHub Actions, Claude Code (agents, skills, MCP) |

## Background

- **QA and DevOps.**
  - Contract QA engineer on an enterprise endpoint-management SaaS: web console and desktop agents on Windows,
    macOS and Linux.
  - Before that, QA and DevOps at H-X Technologies: release QA, server upgrades, a zero-downtime backup migration
    on AWS, and fixes for security findings.
  - Public QA automation work on one real open-source shop:
    [Playwright](https://github.com/BilokinTaras/shop-e2e-playwright) ·
    [pytest](https://github.com/BilokinTaras/shop-api-pytest) ·
    [k6](https://github.com/BilokinTaras/shop-performance-k6) ·
    [the stand](https://github.com/BilokinTaras/shop-sut).
- **Smart-contract security.**
  - Two university theses: detecting vulnerabilities in smart contracts (B.Sc.) and detecting anomalies in
    smart-contract transactions (M.Sc., in progress).
  - Practice audit reports on Cyfrin Updraft training codebases in
    [portfolio-security-review](https://github.com/BilokinTaras/portfolio-security-review).
- **Currently building.** Active Directory attack paths and dynamic mobile testing.

## Certifications and training

- **AWS Certified Solutions Architect – Associate** (SAA-C03), 2026
- Cyfrin Updraft, 2025:
  - Solidity Smart Contract Development
  - Foundry Fundamentals and Advanced Foundry
  - Web3 Wallet Security Basics and Advanced Web3 Wallet Security
  - Blockchain Basics
  - Chainlink Fundamentals
- Cisco Networking Academy, 2023:
  - Introduction to Cybersecurity
  - Network Defense
  - Endpoint Security
  - Cyber Threat Management
  - Networking Basics
  - Networking Devices and Initial Configuration

## Education

- M.Sc. Cybersecurity, Taras Shevchenko National University of Kyiv, 2025 – present
- B.Sc. Cybersecurity, Taras Shevchenko National University of Kyiv, 2022 – 2025
- Software Engineering, Poltava Polytechnic Professional College, 2018 – 2022

## A note on confidentiality

Most of this work was done under statements of work and NDAs. Here I name only public bug bounty programs and
open-source targets, and describe everything else by industry. I give no endpoints, hosts, versions, payloads or
screenshots, and no technical details of anything that a client or program has not cleared for disclosure.
"Reported" means sent to the client or program. "Accepted" and "paid" are used only where that happened. In an
interview I can go deeper on methodology and on any finding the rules allow.

## Contact

[LinkedIn](https://www.linkedin.com/in/taras-bilokin) · bilokintaras@gmail.com · [GitHub](https://github.com/BilokinTaras)
