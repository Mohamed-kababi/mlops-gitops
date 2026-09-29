# mlops-gitops

Deployment manifests for the [MLOps demo](https://github.com/Mohamed-kababi/MLOps). Argo CD syncs `serving/` into the cluster, and it is the only writer to the cluster.

Every commit to `main` here is a deployment, so `git log` is the deployment history.

| File | What it is |
|---|---|
| `serving/rollout.yaml` | The model server under an Argo Rollout: canary 25% → 50% → 100% |
| `serving/model-server-code.yaml` | The FastAPI model server, loading `models:/cancer-classifier/<version>` from MLflow |
| `serving/service.yaml` | The Service in front of stable and canary pods |
| `serving/analysis-templates.yaml` | Canary checks (error rate, p95 latency, prediction mix vs stable) and the Job that moves `champion` in MLflow |

## How a model gets here

The training DAG in the MLOps repo opens a pull request that changes exactly three lines in `serving/rollout.yaml`: `MODEL_VERSION`, `MODEL_URI` and the `model-version` argument. The PR description carries the evaluation scores. Merging it starts a canary.

Manifests always carry a registry version number, never the moving `@champion` alias, so what runs is fully determined by a commit here. Rollback is `git revert`.

## Access

The Airflow token is a fine-grained GitHub token scoped to this repository only, with Contents and Pull requests set to read and write. If it leaks, it can propose deployments here and nothing else.
