# fem-cicd-service — Stable stage

This branch holds the end-of-segment-11 state of the Frontend Masters workshop "Cloud CI/CD with GitHub Actions". The same `dist/` artifact the POC stage shipped is now produced by a multi-contributor pipeline: pull requests are gated by a CI build, dependencies are cached, and the build sequence is extracted into a composite action and a reusable workflow that both the PR check and the deploy share.

## What this branch shows

The build sequence is extracted twice so each variant teaches a different reuse mechanism. The composite action lives at `.github/actions/build-astro/action.yml` and bundles the checkout, Node setup with cache, install, and build into a single step a job can call. The reusable workflow lives at `.github/workflows/_build.yml` and wraps the same sequence as a callable workflow that uploads the `dist/` artifact for a downstream job. Two top-level workflow files consume `_build.yml`: `ci.yml` runs on pull requests and validates the build without deploying, and `deploy.yml` runs on push to `main` and deploys the artifact to S3. Both call the reusable workflow rather than redefining the build inline.

## AWS prerequisites

Create these by hand in the AWS console before pushing. The README uses `<example-bucket>` as a placeholder — substitute your real bucket name where indicated. The Stable stage uses the same AWS topology as POC; no new AWS resources are introduced. CloudFront is deferred to the Enterprise stage.

- One S3 bucket with static-website hosting enabled.
- A bucket policy on `<example-bucket>` granting public read (`s3:GetObject`) on every object. The website endpoint serves traffic directly from S3 — there is no CDN at this stage.
- One IAM user with programmatic access. Attach an inline policy granting `s3:PutObject`, `s3:DeleteObject`, and `s3:ListBucket` on `<example-bucket>` and its contents.
- The IAM user's access key ID and secret access key. You will paste these into GitHub repository secrets in the next section.

## GitHub prerequisites

Configure these on the repository hosting this branch before the first push.

- The repository must be public.
- Two repository secrets: `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`, set to the IAM user's credentials. These are carried forward from the POC stage. Treat them as demo-only and rotate them before the workshop — never reuse production credentials in a teaching repo.
- A branch protection rule on `main`: enable "Require a pull request before merging" and enable "Require status checks to pass before merging" with the `build` status check from `ci.yml` selected as required. This is the gate that makes pull requests the only path to `main`.
- Optional repository secret `ACTIONS_STEP_DEBUG` set to `true`. Used for the segment-8 step-debug demo only; remove or unset it before any production use.
- Replace the `<example-bucket>` placeholder in `.github/workflows/deploy.yml` with the real bucket name before pushing. The workflow will fail otherwise.

## What was deliberately left as the wrong choice

These choices are still wrong on purpose. The list is shorter than the POC stage's because Stable already solves caching, the PR gate, and build reuse. Each remaining item is fixed in the Enterprise stage.

- Long-lived AWS credentials in repository secrets. Anyone with write access to the repo can exfiltrate them. The `enterprise` branch replaces this with OIDC and a short-lived role assumption.
- Marketplace actions are pinned to mutable major-version tags such as `@v4`. A retagged release silently changes what runs in CI. The `enterprise` branch SHA-pins every third-party action.
- No human approval gate on production deploys. Every push to `main` deploys without review. The `enterprise` branch adds a `production` environment with required reviewers and a wait timer.
- No concurrency control. Two simultaneous pushes to `main` race each other to deploy. The `enterprise` branch adds `concurrency:` blocks to serialize deploys.
- Public-read S3 bucket and no CDN. The `enterprise` branch fronts the bucket with CloudFront and locks origin access down with an Origin Access Control policy.

## Linkouts

Workshop and reference documentation lives on `main`; the branches below hold the matching code end-states.

- Master outline: `docs/instructor/OUTLINE.md`.
- Stable stage step-by-step guide: `docs/instructor/STABLE.md`.
- Architecture TDD: `docs/tdd/fem-cicd-workshop-architecture.md`.
- Previous stage reference: run `git checkout poc`.
- Next stage reference: run `git checkout enterprise`.
- Workshop URL: https://frontendmasters.com/workshops/github-actions/.
