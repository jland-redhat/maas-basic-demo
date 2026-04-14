# MaaS basic demo

OpenShift **Models as a Service (MaaS)** demo environment: sample subscriptions, simulator models, and a validation notebook. Use it to show UI flows, rate limits (HTTP 429), and API checks.

## Prerequisites

- OpenShift cluster with MaaS / related operators as required for your environment
- `oc` (or `kubectl`) with a cluster-admin context to apply samples
- Three test users aligned with your demo (see below)

## Deploy the samples

From the repo root, apply the bundled MaaS system (free + premium stacks, LLM inference services, subscriptions, and optional `LLMInferenceServiceConfig` resources under `opendatahub`):

```bash
oc apply -k samples/maas-system
```

Review manifests under `samples/` first if you need to change namespaces or routes.

To deploy only one tier, for example the free stack:

```bash
oc apply -k samples/maas-system/free
```

## Demo users (presenter script)

Use **two browser contexts**: a normal window for end-user personas, and an **incognito (or separate) window** for cluster admin / OpenShift Console work.

| Persona        | Role              | Use for |
|----------------|-------------------|---------|
| **Maria**      | Free user         | Default “basic” path: models, AI Assets, metrics, 429 |
| **Aria**       | Premium user      | Contrasts with free tier where your cluster is set up for it |
| **Bob**        | Enterprise / alt  | Optional second paid or enterprise storyline |

Authorization in the demo is via **OpenShift groups** and tokens (see the validation notebook for API calls).

### Suggested flow

1. **Setup (cluster admin)**  
   In the admin context: OpenShift Console and **Open Data Hub** (or your AI/ML operator UIs as deployed). Confirm groups and any MaaS-related resources match the demo.

2. **Free user path — Maria**  
   - Sign in as Maria.  
   - Open **Models** and show which models she can use.  
   - In **AI Assets** (or your product’s subscription UI), show **subscription** details for the free tier.  
   - Show **metrics** returned for usage.  
   - Exercise traffic until you hit **HTTP 429** (rate limit), matching the configured token limits.

3. **Notebook (optional but clearer for APIs)**  
   Run **`maas-validation-demo.ipynb`** locally with Jupyter: configure `MAAS_BASE` and auth (`OPENSHIFT_TOKEN` or username/password via env), then walk through API key creation, model list, a single completion, and the 429 loop.

4. **Deploy a model (UI)**  
   **Models → Deploy model**  
   - **Model URI:** `hf://sshleifer/tiny-gpt2` — the simulator ignores the weights, but the URI must be a real, fetchable model for the pipeline.  
   - **Environment variable:** `SIM_MODEL` — e.g. set to `enterprise-model` when you want that storyline (adjust to match your simulator / subscription naming).

## Repository layout (short)

| Path | Purpose |
|------|---------|
| `samples/maas-system/` | Kustomize bundles for free/premium MaaS + simulator models |
| `maas-validation-demo.ipynb` | End-user API validation (tokens, completions, 429) |
| `demo-files/` | Ad hoc exports / reference YAML (not applied automatically) |

## Publishing the notebook as static HTML

CI can render the notebook to **GitHub Pages** without executing cells (no cluster calls in the pipeline). See `.github/workflows/deploy-notebook.yml` and enable **Pages → GitHub Actions** in the repository settings.
