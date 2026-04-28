# fem-cicd-service

Example project from Cloud CI/CD with GitHub Actions on Frontend Masters. This repo holds both the workshop's instructor materials (on `main`) and the per-stage completed-solution branches that mirror the workshop's three end-states.

## Repo layout

- **`main`** — instructor materials in [`docs/instructor/`](docs/instructor/) (master outline plus per-stage step-by-step guides) and the architecture TDD in [`docs/tdd/`](docs/tdd/).
- **`poc`** — Stage 1 ("Make it deploy"), end of segments 2–6: sample app plus a single `.github/workflows/deploy.yml`. See TDD §8.1.
- **`stable`** — Stage 2 ("Make it safe to collaborate"), end of segments 8–11: POC content plus a composite action, a reusable workflow, `ci.yml`, and a modified `deploy.yml`. See TDD §8.2.
- **`enterprise`** — Stage 3 ("Make it safe to operate"), end of segments 12–15: Stable content plus OIDC, SHA-pinning, environment, concurrency, and CloudFront. See TDD §8.3.

## Where to start

- **Workshop instructor:** read [`docs/instructor/OUTLINE.md`](docs/instructor/OUTLINE.md) first — its "Branch reference" section is the canonical pointer for the linear-progression relationships and pre-flight checklist.
- **Workshop student following along:** check out the branch matching the stage you want to see (`git checkout poc` / `stable` / `enterprise`) and read its `README.md`.

For architecture context, see [`docs/tdd/fem-cicd-workshop-architecture.md`](docs/tdd/fem-cicd-workshop-architecture.md).

Workshop URL: <https://frontendmasters.com/workshops/github-actions/>.
