# Handoff Context — PoC GitHub Organization Management via IaC

## 1. Background

XBrain/TechX is preparing a broader infrastructure/governance setup. Kai asked for a PoC to manage GitHub Organization teams/members through Infrastructure as Code (IaC).

This PoC is intentionally split into two tasks:

1. **PoC GitHub Organization Members via IaC**
2. **PoC GitHub Organization Teams & Team Membership via IaC**

A separate teammate owns the **repository management via IaC** task, so this PoC should avoid taking ownership of repository lifecycle and team-to-repository permission management unless needed only as a minimal integration check.

The goal is not to over-engineer. First prove the lifecycle and behavior of GitHub Org members/teams with Terraform, including plan/apply/state/drift/import behavior.

---

## 2. Recommended Test Environment

Do **not** start by applying this against the real company GitHub Organization.

Preferred setup:

- A **sandbox/test GitHub Organization** where the tester has sufficient administrative permissions.
- A dedicated Git repository to hold the Terraform PoC code, for example:
  - `github-org-iac-poc`
- At least 2 GitHub test users if possible:
  - one org owner/admin used to run Terraform
  - one normal test account used as the member being invited/removed
- Optional second test member to validate multiple memberships and team roles.

A normal personal repository alone is **not enough** to validate organization membership/team lifecycle. The resources being tested are organization-level resources.

The test repository is mainly used for:
- storing Terraform code,
- versioning changes,
- creating commits/PRs,
- keeping evidence of `terraform plan/apply`,
- documenting results.

---

## 3. Tooling

Primary tool:

- Terraform
- GitHub Provider: `integrations/github`

Codex should verify the latest compatible provider documentation before implementation.

Expected provider resources to investigate:

- `github_membership`
  - manages user membership in a GitHub Organization
- `github_team`
  - manages GitHub Organization teams
- `github_team_membership`
  - manages the relationship between an existing org member and a team

Repository-related resources such as `github_team_repository` are outside the main ownership of this PoC because repository IaC is assigned separately.

---

## 4. Task A — PoC GitHub Organization Members via IaC

### Goal

Prove that GitHub Organization member lifecycle can be managed declaratively through Terraform rather than manually through the GitHub UI.

### Research

Determine:

- exact Terraform resource(s) used for organization membership,
- supported member roles,
- authentication methods supported by the provider,
- exact permissions/scopes required,
- invitation behavior when a user has not yet accepted the organization invite,
- behavior when removing a member,
- import behavior for an existing organization member,
- state/drift behavior.

Do not assume permissions/scopes; verify them against current GitHub Provider/GitHub API documentation.

### PoC flows

Implement and test:

1. Add/invite a GitHub user to the Organization.
2. Verify the invitation/member state on GitHub.
3. Change the member role if supported by the resource and available permissions.
4. Remove the member by changing IaC configuration.
5. Run `terraform plan` after each change and confirm expected diff.
6. Perform one manual change in GitHub UI, then run `terraform plan` and observe drift behavior.
7. Import an already-existing member into Terraform state if supported and validate that Terraform does not unexpectedly destroy/recreate it.

### Expected evidence

Capture:

- Terraform code,
- `terraform init`,
- `terraform validate`,
- `terraform plan`,
- `terraform apply`,
- relevant GitHub UI/API result,
- drift test result,
- import test result,
- limitations/edge cases.

### Expected output

A short conclusion answering:

> Can GitHub Organization member lifecycle be safely and predictably managed through Terraform?

---

## 5. Task B — PoC GitHub Organization Teams & Team Membership via IaC

### Goal

Prove that GitHub Organization teams and team membership can be managed declaratively through Terraform.

### Research

Determine:

- exact Terraform resource(s) used for organization teams,
- supported team properties,
- supported team membership roles,
- parent/child team support,
- privacy/visibility behavior,
- import support,
- drift behavior,
- delete/update behavior,
- dependency between org membership and team membership.

### PoC flows

Implement and test:

1. Create a team.
2. Update team properties.
3. Add an existing organization member to the team.
4. Change the member's team role where supported.
5. Add the same user to multiple teams if useful for validation.
6. Remove a member from a team.
7. Delete a team.
8. Perform a manual UI change and verify whether `terraform plan` detects drift.
9. Import an existing team into Terraform state if supported.
10. Validate dependencies so Terraform does not try to add a user to a team before the organization membership exists.

### Expected evidence

Capture:

- Terraform code,
- plan/apply output,
- team visible in GitHub Org,
- team membership visible in GitHub Org,
- drift detection,
- import result,
- lifecycle/dependency behavior.

### Expected output

A short conclusion answering:

> Can GitHub Organization team and team-membership lifecycle be safely and predictably managed through Terraform?

---

## 6. Proposed Repository Structure

Start simple.

```text
github-org-iac-poc/
├── README.md
├── versions.tf
├── provider.tf
├── variables.tf
├── members.tf
├── teams.tf
├── terraform.tfvars.example
├── .gitignore
└── docs/
    ├── member-poc-results.md
    └── team-poc-results.md
```

Do not create a complex module hierarchy until the basic PoC works.

---

## 7. Authentication / Secrets Guardrails

Do not hard-code credentials.

Use environment variables or another secure local mechanism supported by the Terraform GitHub Provider.

Never commit:

- GitHub PAT/token,
- GitHub App private key,
- Terraform state containing sensitive material,
- `.tfvars` containing secrets.

At minimum add appropriate entries to `.gitignore`.

For the first PoC, choose the simplest authentication method that is safe for a sandbox environment, but document what should change for a production implementation.

Codex must explicitly verify:
- required GitHub permission/scopes,
- whether the chosen token/app can manage org members,
- whether it can manage teams,
- whether organization-owner privileges are required for the tested operations.

---

## 8. Suggested Terraform Starting Point

Do not copy this blindly; verify resource arguments against the current provider docs.

```hcl
terraform {
  required_providers {
    github = {
      source = "integrations/github"
    }
  }
}

provider "github" {
  owner = var.github_org
}
```

Conceptually:

```hcl
resource "github_membership" "test_user" {
  username = var.test_username
  role     = "member"
}
```

```hcl
resource "github_team" "devops" {
  name        = "poc-devops"
  description = "PoC team managed by Terraform"
  privacy     = "closed"
}
```

```hcl
resource "github_team_membership" "test_user_devops" {
  team_id  = github_team.devops.id
  username = github_membership.test_user.username
  role     = "member"
}
```

Use explicit dependencies only if Terraform cannot infer them naturally.

---

## 9. Definition of Done

The PoC is complete when:

### Members
- Terraform can add/invite a test user to the sandbox Organization.
- Terraform can update supported membership role/state.
- Terraform can remove the member.
- `terraform plan` reflects config changes correctly.
- At least one manual-change/drift test is documented.
- Import of an existing member is tested if supported.

### Teams
- Terraform can create/update/delete a test team.
- Terraform can add/remove an existing Org member to/from the team.
- Supported team role changes are tested.
- At least one manual-change/drift test is documented.
- Import of an existing team is tested if supported.

### Documentation
- README explains prerequisites, auth, commands, permissions, risks, and cleanup.
- Evidence/results are recorded under `docs/`.
- No secrets are committed.
- All PoC-created test resources can be cleaned up safely.

---

## 10. Important Scope Boundary

Do not expand this PoC into the separate repository-management task.

This PoC owns:

```text
Organization
├── Members
└── Teams
    └── Team Membership
```

The other task owns:

```text
Repositories
├── Repository lifecycle/settings
└── Repository access/policies
```

The two PoCs may later be integrated into:

```text
User
  ↓
Organization Membership
  ↓
Team Membership
  ↓
Repository Permission
```

but keep the first implementation boundaries clear.

---

## 11. First Actions for Codex

1. Read current Terraform GitHub Provider documentation for:
   - provider authentication,
   - `github_membership`,
   - `github_team`,
   - `github_team_membership`.
2. Confirm exact permissions/scopes and role values.
3. Generate the minimal repository structure above.
4. Implement only the member PoC first.
5. Run format/validate.
6. Before any destructive/apply step, show the expected resource changes.
7. After member PoC works, add the team PoC.
8. Produce a short result document with:
   - what worked,
   - what did not,
   - limitations,
   - recommendation for production usage.

Keep the solution minimal and auditable. Avoid adding CI/CD, remote state, modules, custom wrappers, or policy engines until the basic PoC is proven.
