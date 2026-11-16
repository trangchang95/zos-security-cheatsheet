# z/OS security cheat sheet, in plain words

A short, practical reference for people who need to understand how access is controlled on an IBM z/OS mainframe with RACF. It covers concepts only. Every example uses made-up names. No employer, system or data is referenced.

> Not official documentation. For anything that matters, check IBM's *z/OS Security Server RACF* manuals.

## The one question

Every request on z/OS comes down to: **who is asking, what do they want to touch, and what are they allowed to do?** RACF answers it with three building blocks.

| Block | Meaning | Example |
|---|---|---|
| User ID | Who is asking | `ALICE01` |
| Group | A set of users sharing access | `PAYDEV` |
| Profile | A rule attached to a resource | `PAYROLL.DATA.**` |

## Access levels (lowest to highest)

| Level | What it allows |
|---|---|
| NONE | Nothing |
| READ | Read only |
| UPDATE | Read and write |
| CONTROL | UPDATE plus extra control on VSAM data |
| ALTER | Full control, including delete and rename |

## UACC and the access list

- **UACC** (universal access) is what everyone gets by default on a profile. The safe default is `NONE`.
- The **access list** gives named users or groups a specific level.
- Rule of thumb: UACC low, access list explicit, groups over individuals.

## Profiles and resource classes

- **Discrete profile**: protects exactly one resource.
- **Generic profile**: protects many resources by pattern (`*`, `**`, `%`).
- **Resource class**: the type of thing protected, such as `DATASET`, `FACILITY`, `SURROGAT`, `TERMINAL`. A class must be active for its profiles to be enforced.

## User attributes worth knowing

| Attribute | Power |
|---|---|
| SPECIAL | Manage RACF profiles |
| OPERATIONS | Broad access to datasets and some resources |
| AUDITOR | Control and read audit settings |
| REVOKE | ID is disabled |

Handing these out widely is a common finding in reviews.

## Everyday commands (sketch)

```
ADDUSER  ALICE01 NAME('Alice Example') DFLTGRP(PAYDEV)
ADDGROUP PAYDEV
CONNECT  ALICE01 GROUP(PAYDEV)
ADDSD    'PAYROLL.DATA.**' UACC(NONE)
PERMIT   'PAYROLL.DATA.**' ID(PAYDEV) ACCESS(READ)
LISTUSER ALICE01
SEARCH   CLASS(USER)
SETROPTS LIST
```

All names above are invented examples.

## Where reviews usually find problems

Based on publicly available practitioner material on common RACF weaknesses:

- Universal access set too high on profiles
- Special authorities (SPECIAL, OPERATIONS, AUDITOR) given too widely
- Stale access lists and IDs left after people transfer or leave
- Resource classes not active, or no catch-all profile
- Weak logon controls, such as weak password rules and no MFA
- Gaps in audit logging and short log retention
- Started-task and batch IDs with more privilege than needed

## Access review checklist

1. List who holds SPECIAL, OPERATIONS, AUDITOR. Is each still needed?
2. List IDs with no recent logon. Revoke or remove.
3. For sensitive datasets, compare the access list with the real need.
4. Confirm UACC is NONE unless there is a written reason.
5. Check that logging covers privileged use.
6. Record the owner who approved each access.

## Glossary

- **IAM**: identity and access management.
- **Least privilege**: give only the access needed for the task.
- **Recertification**: owners periodically confirm who should keep access.

## Status

Work in progress. Planned next: a small Python parser that reads *synthetic* RACF-style reports.

Author: Trang Chang
