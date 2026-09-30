# Task B results: teams and team membership

Date: **2026-09-30**. Scope: local scaffold and validation. **Live GitHub PoC is not yet completed.** No sandbox organization/login or test-member configuration was supplied. No live plan, apply, import, manual drift change, or destructive operation was performed.

## Local evidence

| Check | Result |
| --- | --- |
| Current provider documentation | Verified official Registry/provider and GitHub API docs; [research and sources](provider-research.md) |
| Terraform runtime | 1.16.4, Windows amd64; downloaded into ignored `.tools/` and SHA256 checked against HashiCorp release checksums |
| Provider initialization | Passed: `terraform init -backend=false -input=false`; integrations/github 6.13.0, partner signature verified; `.terraform.lock.hcl` generated |
| Formatting | Passed: `terraform fmt -check -recursive` |
| Schema validation | Passed: `terraform validate` |
| Mocked safety tests | Passed: `terraform test -no-color` reported **12 passed, 0 failed**; every run uses `command = plan` and a mocked GitHub provider |
| Git exclusions | Verified local tfvars, state/backups, saved plans, token/key files, tooling, and raw local evidence are ignored; source example and lock file remain trackable |
| Live plan/apply | Not run; sandbox details and authenticated session needed |

The Terraform ZIP SHA256 was `5c736ed6b0f13e98bc5029fe159574cca33634c3477466c56442c28b513a7054`. Tool downloads and Terraform init did not mutate a GitHub organization. Mocked tests verify Terraform behavior only and are not live lifecycle evidence.

The 12 tests cover active/pending membership, both owner-role outcomes, distinct addresses for a shared username across teams with one org lookup, invalid roles/privacy, secret parents/children, missing parents, unsupported deeper nesting, and an empty desired configuration. [Test source](../tests/task-b.tftest.hcl).

## Live run record

Fill these fields during the sandbox run. Use the [README procedure](../README.md) and attach redacted evidence rather than credentials, state, or saved plan files.

- Sandbox organization: **not supplied**
- Runner login / authorization method and permission names (no token): **not supplied**
- Ordinary active test member: **not supplied**
- Tested commit/configuration revision: **pending**
- Team IDs/slugs and original import properties: **pending**

For every applied step, record the date, plan summary, reviewer/approval, redacted plan/apply output, UI/API observation, and subsequent no-change plan. A failed step should include the error and recovery without marking later dependent steps successful.

| Flow | Expected result | Actual result / evidence |
| --- | --- | --- |
| Create team + direct membership | Example: 2 adds; team/member visible; follow-up plan has no changes | Not run |
| Update description/name/notifications | In-place update; numeric team ID preserved | Not run |
| Privacy change on standalone team | `closed`/`secret` update; expected visibility | Not run |
| Add existing active member | One new direct membership; org lookup precedes membership operation | Not run |
| Change ordinary member role | `member` -> `maintainer` -> `member`; stable org membership | Not run |
| Same user in two independent teams | Two distinct direct memberships | Not run |
| Parent/child creation | Child created with managed parent's ID; both closed | Not run |
| Remove direct membership | One membership destroy; org membership remains active | Not run |
| Delete disposable team | Team/memberships removed; descendants/access reviewed first | Not run |
| Manual team drift | Normal plan detects changed description and proposes restoration | Not run |
| Manual membership drift | Normal plan detects changed role or removal | Not run |
| Team import | Reviewed import only, no unintended update/replacement; then no-change plan | Not run |
| Direct membership import | Reviewed pair import only; then no-change plan | Not run |
| Pending invitee / outsider | Plan fails safely; no invitation is sent by this configuration | Not run against GitHub |
| Owner role guard | Owner requested as member is rejected; maintainer is allowed | Not run against GitHub |
| Final cleanup | Only approved PoC resources removed; state empty and org members retained | Not run |

## Limitations and conclusion

Current docs and provider source support the required team lifecycle. This implementation adds read-only active-org-member checks, owner-role validation, explicit parent/team dependencies, and additive direct membership management. Its configuration can be evaluated locally without credentials.

**Can the lifecycle be safely and predictably managed?** It appears feasible within these boundaries, but the live PoC has not established that conclusion yet. Authentication, GitHub-side lifecycle, drift, import, and cleanup remain unverified until the table above is completed.

Plan-time membership checks have a race with later org changes; regenerate plans promptly. Extra unconfigured memberships remain unmanaged, inherited access may persist after direct removal, parent deletion can cascade beyond state, and root/child address changes require deliberate state migration. IdP-synchronized teams and organization-member lifecycle are outside scope. See [verified behavior](provider-research.md).

Production recommendation: finish the sandbox evidence first, then evaluate GitHub App credentials, protected shared state, review controls, and an explicit ownership boundary with the separate repository IaC effort.
