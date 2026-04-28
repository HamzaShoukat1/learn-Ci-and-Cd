---
marp: true
theme: default
class:
  - invert
  - lead
last_updated: 2026-04-27
---

<style>
h1 {
    font-size: 48px;
}
section {
    background: black;
}
</style>

# Cloud CI/CD with GitHub Actions

Build CI/CD pipelines for a real frontend, three stages of maturity.

---

# Introduction to Teacher

![hello](https://media.giphy.com/media/3ornk57KwDXf81rjWM/giphy.gif)

---

## Erik Reinert aka "Blackglasses"

- Senior software engineer
- Content creator (@TheAltF4Stream)
- Diagram & flowchart artist
- Habitual problem solver

---

## Work Experience

- Started with frontend (2+ years)
- Followed curiosity to backend (2+ years)
- Continued curiosity to fullstack (2+ years)
- Found passion in DevOps & Platform Engineering (4+ years - current)

---

## I build things on the internet

- Twitch: https://www.twitch.tv/thealtf4stream
- YouTube: https://www.youtube.com/thealtf4stream
- Twitter: https://www.x.com/thealtf4stream
- Blog: https://altf4.blog

---

## Existing Courses

- Introduction to DevOps for Developers
- Enterprise Cloud Infrastructure
- Introduction to Backend Architectures
- Fullstack Deployment: From Containers to Production AWS

---

## Introduction to DevOps for Developers

Take your first steps into DevOps guided from the perspective of a developer! Improve software teams' ability to build and ship software reliably.

---

## Enterprise Cloud Infrastructure

Learn to set up large-scale systems with GitOps and optimized CI/CD workflows. And see strategies to standardize your organization's approach to AWS resource management and dynamic cloud orchestration.

---

# Course Introduction

![welcome](https://media1.giphy.com/media/v1.Y2lkPTc5MGI3NjExY21yOTZrcHhtcHZzdm40ZjZvc2Eyczd4N3BrZWFkbTUxcGliNGJuNiZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/ASd0Ukj0y3qMM/giphy.gif)

---

## Goals in this course

- Ship a frontend with GitHub Actions
- Recognize three stages of CI/CD maturity
- Know when to escalate from "it works" to "it's safe"
- Build a pipeline you'd be okay handing to a security review

---

## Pre-requisites for this course

- A GitHub account (Free plan is fine)
- Basic Astro / Node.js familiarity
- AWS account on the free tier with admin access
- A public repo prepared for the workshop
- Node 22.22 installed locally

---

## How this workshop runs

- The slides set up the WHY
- The terminal does ALL the code
- You can follow along live, or read the branches later

---

## Reference branches

- POC end state -> `git checkout poc`
- Stable end state -> `git checkout stable`
- Enterprise end state -> `git checkout enterprise`

---

## Working branch

- Workshop -> `git checkout feature/initial-implementation`

---

# Course Structure

![structure](https://media3.giphy.com/media/v1.Y2lkPTc5MGI3NjExanFzZXByMXRjZGszZDJydGx2dGxkMnJweWduOXpkd3c5dXFhOGI4MyZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/l0IylOPCNkiqOgMyA/giphy.gif)

---

## Three Stages

- POC — "Just get it deploying!"
- Stable — "Make it safe to collaborate!"
- Enterprise — "Make it safe to operate!"

---

## Same artifact, three pipelines

- One trivial Astro `dist/` rides three increasingly mature pipelines
- The diff between pipelines IS the workshop
- By 4:30 you can articulate which pipeline fits which moment

---

## POC

> "Just get it deploying!"

---

## Stable

> "Make it safe to collaborate!"

---

## Enterprise

> "Make it safe to operate!"

---

## The day shape

- Morning: POC (segments 2-6)
- Lunch (segment 7)
- Early afternoon: Stable (segments 8-11)
- Late afternoon: Enterprise (segments 12-15)
- Wrap-up at 4:30

---

# Stage 1 — POC

![chaos](https://media.giphy.com/media/13HgwGsXF0aiGY/giphy.gif)

---

## POC

> "Just get it deploying!"

---

## Phase Scenario

- We have working source code
- We need bytes in front of a user this week
- We are willing to do the wrong thing on purpose
- We will name what's wrong as we go

---

## Phase Goals

- One workflow file in the repo
- Build the Astro site in CI
- Ship `dist/` to S3 on push to `main`

---

## Phase Requirements

- A clean GitHub repo on `main` only
- An S3 bucket with static website hosting (public-read)
- An IAM user with programmatic S3 access
- Repo secrets ready to receive the access key pair

---

## End-of-POC pipeline

```mermaid
flowchart LR
    Push["push to main"] --> Build["build job<br/>checkout + setup-node<br/>npm ci + npm run build<br/>upload-artifact"]
    Build --> Deploy["deploy job<br/>download-artifact<br/>configure-aws-credentials (IAM key)<br/>aws s3 sync"]
    Deploy --> S3[("S3 bucket<br/>public-read")]
```

---

## Start building!

---

## Segment 2 — Your First Workflow

- Workflows are YAML in `.github/workflows/`
- No separate CI server to configure
- The smallest workflow is a single `echo`
- See one run end-to-end before naming concepts

---

## Why segment 2 stays tiny

- Names land in segments 3 and 4
- Indentation errors are the #1 reason workflows "don't run"
- If GitHub doesn't see the file, suspect YAML before suspecting GitHub

---

## Segment 3 — Triggers & Runners

- `on:` is WHEN
- `runs-on:` is WHERE
- Each job runs on a fresh VM — no shared state between jobs
- Self-hosted exists; we revisit in segment 15

---

## Triggers we actually use

- `push: branches: [main]` — narrowed from "any push"
- `workflow_dispatch` — the manual button
- `pull_request` — coming in Stable
- Everything else: read the docs when you need it

---

## Segment 4 — Contexts & Expressions

- `${{ ... }}` is the only place YAML has logic
- Read who triggered the run, on what ref, from what event
- Gate steps with `if:`
- If you need real logic, write a script — not more `${{ }}`

---

## The contexts that matter today

- `github` — actor, ref, event_name, sha
- `runner` — OS, temp dir
- `secrets` — encrypted values (NOT readable in `if:`)
- `vars` — repo-level variables (readable everywhere)

---

## Segment 5 — Building a CI Pipeline

- Three steps every Node project needs: checkout, setup-node, install + build
- Then add AWS credentials and `aws s3 sync`
- The build IS the test for a static site

---

## Yes, we are doing the wrong thing on purpose

- IAM user access key pasted into repo secrets
- Anyone with write access to the repo can exfiltrate it
- The screenshare recording is permanent
- Enterprise (segment 13) fixes this — make the wrong way concrete first

---

## NO LONG-LIVED CREDENTIALS

> Long-lived AWS keys in repo secrets is the original sin of CI/CD.
> We're committing it on stage so the OIDC payoff lands later.

---

## Segment 6 — Job Dependencies & Artifacts

- Real pipelines split build from deploy
- Each job runs on a fresh runner — no shared filesystem
- `upload-artifact` / `download-artifact` move the bytes
- `needs:` declares the ordering

---

## Why split build and deploy?

- The artifact becomes a first-class object
- You can redeploy without rebuilding
- Multiple deploy targets reuse the same build
- A failed build short-circuits before any deploy fires

---

## End-of-POC recap

- Push to `main` builds and deploys to S3
- Two jobs, one artifact, one workflow file
- Long-lived credentials, no review, no caching, no concurrency
- We named the four problems Stable will solve

---

## Phase Changes

- added `.github/workflows/deploy.yml`
- added repo secrets for AWS credentials
- added an S3 bucket and bucket policy out-of-band
- one workflow, two jobs, one artifact

---

## Phase Pros

- Push and ship — a developer can read it
- One workflow file is auditable end-to-end
- Zero infrastructure beyond a bucket
- Fastest path to "users see the site"

---

## Phase Cons

- Long-lived AWS keys in repo secrets
- No review — every push deploys
- No caching — install every run
- No concurrency — last writer wins, non-deterministically

---

# Stage 2 — Stable

![calm](https://media.giphy.com/media/l4FGuhL4U2pJgLp32/giphy.gif)

---

## Stable

> "Make it safe to collaborate!"

---

## Phase Scenario

- POC is shipping
- Now multiple people want to commit
- We need a PR gate before code reaches `main`
- We need workflow code we are not embarrassed to read

---

## Phase Goals

- Cache npm so installs finish in seconds
- Validate pull requests before they merge
- Extract the build sequence to ONE place
- Protect `main` so direct pushes are blocked

---

## Phase Requirements

- POC pipeline is green on `main`
- A feature branch ready for the PR demo
- `ACTIONS_STEP_DEBUG` slot ready for the debugging demo
- Branch-protection settings tab open in the browser

---

## End-of-Stable pipeline

```mermaid
flowchart LR
    PR["pull_request"] --> CI["ci.yml<br/>uses _build.yml"]
    Push["push: main"] --> Deploy["deploy.yml<br/>build job uses _build.yml<br/>deploy job aws s3 sync"]
    CI -. required check .-> MainBranch[("main protected")]
    MainBranch --> Push
    Deploy --> S3[("S3 bucket<br/>public-read")]
    CI -.-> Reusable["_build.yml"]
    Deploy -.-> Reusable
    Reusable -.-> Composite["build-astro composite"]
```

---

## Start building!

---

## Segment 8 — Caching & Debugging

- Cache is the highest-leverage change you can make
- `setup-node` with `cache: 'npm'` — short and hard to misconfigure
- `actions/cache` — when you need something setup-node doesn't know about
- `ACTIONS_STEP_DEBUG=true` is the equivalent of `set -x`

---

## Yes, this fails on purpose

- We will deliberately mistype the cache key
- CI goes red — we read the log
- A red CI is the signal the pipeline exists for, not a problem to hide
- Recognize the failure pattern when you meet it for real

---

## Segment 9 — Marketplace & Composite Actions

- Marketplace is a directory, not a registry
- Treat actions like third-party deps
- A composite action wraps a step sequence inside a single job
- It can have inputs and outputs, but no `runs-on:` of its own

---

## Reading a marketplace action

- Publisher — first-party (`actions/`, `aws-actions/`) or community
- Release cadence — three years quiet is a risk
- Star count — not quality, but blast radius
- Security advisories tab — has it been audited

---

## Why a composite for the build sequence

- setup-node + install + build + upload-artifact = one logical unit
- Future workflows in this repo will call the same composite
- Local composites can't bootstrap their own checkout
- Checkout stays in the calling job — every time

---

## Segment 10 — Reusable Workflows

- A reusable workflow wraps a whole job (or jobs)
- Triggered by `on: workflow_call`
- Runs as its OWN job with its OWN runner
- Composite is step-level; reusable is job-level

---

## What branch protection actually does

- Required checks turn CI from advisory into enforced
- `main` no longer accepts direct pushes
- Merging the PR is the only way bytes reach `main`
- This is the moment "multi-contributor" stops being theoretical

---

## Segment 11 — Composite vs. Reusable vs. Custom

- Step sequence in one job? Composite.
- Whole job, possibly across repos? Reusable workflow.
- Logic that doesn't fit YAML? Custom JS or Docker action.
- Three mechanisms, three different questions.

---

## Default to composite

- Simpler to author
- Logs appear inline in the calling job
- Zero runtime overhead
- Reach for reusable when the unit is GENUINELY a job

---

## End-of-Stable recap

- PRs gate `main` via branch protection
- Builds are cached and finish in seconds
- The build sequence lives in one place, called from two workflows
- Long-lived keys are still there. We'll fix it next.

---

## Phase Changes

- added `.github/actions/build-astro/action.yml`
- added `.github/workflows/_build.yml`
- added `.github/workflows/ci.yml`
- modified `deploy.yml` to call the reusable
- enabled branch protection on `main`

---

## Phase Pros

- Bad code can't reach `main` without review
- Builds are fast (npm cache)
- One source of truth for the build sequence
- The pipeline is portfolio-grade

---

## Phase Cons

- Long-lived AWS keys still in repo secrets
- Deploys still automatic on merge — no human gate
- Actions pinned to mutable major-version tags
- S3 bucket still public-read, no CDN

---

# Stage 3 — Enterprise

![serious](https://media.giphy.com/media/3o7TKDEq4bUZbFNVjW/giphy.gif)

---

## Enterprise

> "Make it safe to operate!"

---

## Phase Scenario

- The pipeline works and the team trusts it
- Now a security review is on the calendar
- Long-lived keys, public buckets, mutable tags — all findings
- We have to make the pipeline reviewable

---

## Phase Goals

- Replace long-lived AWS keys with OIDC
- Gate the deploy on a human
- Pin actions to commit SHAs
- Front S3 with CloudFront + OAC
- Add concurrency control

---

## Phase Requirements

- IAM role pre-created with OIDC trust policy
- GitHub OIDC provider already trusted in AWS
- CloudFront distribution pre-staged (Disabled)
- S3 bucket policy ready to flip to OAC-only

---

## End-of-Enterprise pipeline

```mermaid
flowchart LR
    PR["pull_request"] --> CI["ci.yml<br/>SHA-pinned<br/>concurrency: pr-#"]
    Push["push: main"] --> Deploy["deploy.yml<br/>environment: production<br/>concurrency: prod-deploy<br/>id-token: write"]
    CI -. required check .-> MainBranch[("main protected")]
    MainBranch --> Push
    Deploy --> OIDC{{"OIDC token exchange<br/>sts:AssumeRoleWithWebIdentity"}}
    OIDC --> Role["IAM Role<br/>least-privilege"]
    Role --> S3[("S3 (private)")]
    Role --> CF[("CloudFront<br/>OAC + invalidation")]
    S3 --> CF
    CF --> Users(("End users"))
```

---

## Start building!

---

## Segment 12 — Environments & Protection Rules

- An environment is a named bundle of rules and secrets
- Required reviewers — pause until a human approves
- Wait timer — the "are you SURE?" guardrail
- Free for public repos; paid plan for private

---

## Five seconds of dead air

> 30 minutes of count-down on stage = unwatchable.
> 1 minute = teachable.
> The wait timer is not a queue — it's a brake.

---

## Why environments come first in Enterprise

- The `production` environment is the principal
- The OIDC trust policy will bind to that principal
- Segment 12 establishes the binding target
- Segment 13 wires up the binding

---

## Segment 13 — OIDC & Cloud Authorization

- Short-lived credentials minted at runtime
- The keys never sat in GitHub
- The trust policy is principal-level least privilege
- A leaked workflow file from another repo can't mint our credentials

---

## How OIDC works (mental model)

- GitHub mints a signed token describing this run
- AWS verifies the signature and reads the claims
- Claims must match the IAM trust policy
- AWS exchanges the token for credentials that expire in an hour

---

## NO LONG-LIVED CREDENTIALS — FINALLY GONE

> The `AWS_ACCESS_KEY_ID` secret is deleted on stage.
> The `AWS_SECRET_ACCESS_KEY` secret is deleted on stage.
> The list is empty. The leak surface no longer exists.

---

## CloudFront + OAC closes the loop

- S3 flips from public-read to private
- Only the OAC service principal can read objects
- CloudFront becomes the public surface
- The deploy job invalidates the cache after sync

---

## OWASP CICD-SEC closed in segment 13

- CICD-SEC-2 — Insufficient flow control mechanisms
- CICD-SEC-6 — Insufficient credential hygiene

---

## Segment 14 — Hardening Your Workflows

- SHA-pinning freezes the action at the code we reviewed
- Deny-all permissions is auditable
- A reader of the workflow can answer "what can this do?" without GitHub's defaults table
- Mechanical work — every `uses:` re-pinned with a comment

---

## Mutable tag = Russian roulette

> A publisher's `@v4` tag could be re-pointed to malicious code overnight.
> Every workflow on the planet pinned to `@v4` re-fetches that code on its next run.
> The 40-character SHA freezes you on the version you reviewed.

---

## Where SHAs live

- `_build.yml`
- `ci.yml`
- `deploy.yml`
- `build-astro/action.yml` (don't forget the composite!)
- Each `uses:` gets a SHA and a `# vX.Y.Z` comment

---

## Permissions: deny-all by default

- Workflow level: `permissions: {}`
- Each job grants ONLY what it needs
- `contents: read` for jobs that checkout
- `id-token: write` ONLY on the deploy job

---

## OWASP CICD-SEC closed in segment 14

- CICD-SEC-1 — Insufficient identity & access management
- CICD-SEC-3 — Third-party action integrity

---

## Segment 15 — Concurrency & Self-Hosted Runners

- Concurrency groups serialize where order matters
- Concurrency groups parallelize where it doesn't
- PR validation — cancel superseded runs
- Production deploys — queue, never cancel

---

## Why deploys never cancel

- A deploy mid-sync leaves S3 and CloudFront inconsistent
- You don't want to find out at 4 PM Friday that half your assets are old
- Cancellation is safe when work is idempotent
- It's unsafe when work has side effects partway through

---

## Self-hosted runners — discussion only

- Capability mentioned, never demoed live
- Three legitimate reasons: VPC-internal, licensed tooling, very large jobs
- Three operational concerns: patching, autoscaling, credential exposure
- Office hours / FEM platform comments for more

---

## End-of-Enterprise recap

- OIDC removed long-lived credentials
- `production` environment gates the deploy
- SHA-pinning freezes third-party code
- Concurrency controls prevent races
- The pipeline is now reviewable

---

## Phase Changes

- added `production` environment with reviewers + wait timer
- replaced AWS keys with OIDC `role-to-assume`
- enabled CloudFront with OAC, made S3 private
- pinned every `uses:` to a 40-char SHA
- added deny-all permissions plus per-job grants
- added concurrency groups to both workflows

---

## Phase Pros

- No long-lived credentials anywhere
- Human in the loop before production changes
- Third-party action drift is impossible
- Production deploys are serialized
- Workflow files are reviewable by security

---

## Phase Cons

- More moving parts to operate
- AWS-side setup is a real prerequisite
- SHA bumps need Dependabot or human discipline
- The team has to actually use the reviewer gate

---

# Course Recap

![recap](https://media1.giphy.com/media/v1.Y2lkPTc5MGI3NjExcnZjN3JqaDA4Y3VpbWwxcTQwMmdyMGlpZDg0bGR4Mmx6azZpY2lpZiZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/3o6ZtlYXUF93rBxr1K/giphy.gif)

---

## What did we do?

- Shipped the same `dist/` artifact through three pipelines
- Watched a workflow grow from `echo` to OIDC + CloudFront
- Named what was wrong at every stage before fixing it
- Closed four OWASP CICD Top 10 findings on stage

---

## What did we learn?

- How to scope a CI/CD change to the moment the team is in
- How to recognize a deploy ready for a security review
- How to choose between composite, reusable, and custom actions
- How to read a marketplace action like a third-party dep

---

## Three-stage progression

- POC — push and ship, do the wrong thing knowingly
- Stable — PR-gated, cached, DRY, branch-protected
- Enterprise — OIDC, environments, SHAs, concurrency, CDN

---

## OWASP CICD Top 10 closed today

- CICD-SEC-1 — Insufficient identity & access management
- CICD-SEC-2 — Insufficient flow control
- CICD-SEC-3 — Third-party action integrity
- CICD-SEC-6 — Insufficient credential hygiene

---

## Thanks for watching

---

## I build things on the internet

- Github (personal): https://github.com/erikreinert
- Github (company): https://github.com/ALT-F4-LLC
- Twitch: https://www.twitch.tv/thealtf4stream
- YouTube: https://www.youtube.com/thealtf4stream
- Twitter: https://www.x.com/thealtf4stream
- Blog: https://altf4.blog
