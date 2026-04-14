# MaaS basic demo

OpenShift **Models as a Service (MaaS)** demo: sample subscriptions, simulator models, and Jupyter notebooks for UI flows, rate limits (HTTP 429), and API checks.

## Prerequisites

- OpenShift cluster with MaaS (and related operators) as required for your environment
- `oc` (or `kubectl`) with a cluster-admin context to apply samples
- Three test users aligned with your demo (see [Demo users](#demo-users-presenter-script))

## OpenShift CLI and console (RHOAI demos)

**You do not need `oc` inside the Jupyter kernel.** RHOAI notebook images usually omit the OpenShift CLI. Run **`oc`** on your **workstation** (or any host) where you are logged into the cluster, or use the **OpenShift web console** to copy a login token.

**Useful commands** (after `oc login` or with a valid kubeconfig):

```bash
oc whoami                    # current OpenShift user
oc whoami -t                 # bearer token (same idea as “Copy login command” in the console)
oc whoami --show-console     # web console URL (OpenShift 4.10+)
oc cluster-info              # Kubernetes API endpoint
```

**MaaS gateway URL (`MAAS_BASE`):** Use the public HTTPS origin for your MaaS gateway (the host you use for `/maas-api/...` in the browser). Your team may document it as a route; you can also list routes with `oc get route -A` if you know the namespace.

**Notebooks:** The demo notebooks include a **Demo quick swap** section so you can paste `MAAS_BASE` and a token or API key in one cell and re-run setup. **`ExternalModelsDemo.ipynb`** uses the same API-key pattern and defaults to two gateway **`/v1/chat/completions`** routes (GPT-4o and Claude); switch with `ACTIVE_EXTERNAL` (no catalog flow).

## Deploy the samples

From the repo root, apply the bundled MaaS system (free and premium stacks, LLM inference services, subscriptions, and optional `LLMInferenceServiceConfig` resources under `opendatahub`):

```bash
oc apply -k samples/maas-system
```

Review manifests under `samples/` before changing namespaces or routes.

Single tier (example — free stack):

```bash
oc apply -k samples/maas-system/free
```

## Demo users (presenter script)

Use **two browser contexts**: a normal window for end-user personas, and an **incognito (or separate) window** for cluster admin / OpenShift Console work.

| Persona   | Role             | Use for |
|-----------|------------------|---------|
| **Maria** | Free user        | Default path: models, AI Assets, metrics, HTTP 429 |
| **Aria**  | Premium user     | Contrast with free tier where your cluster supports it |
| **Bob**   | Enterprise / alt | Optional second paid or enterprise storyline |

Authorization is via **OpenShift groups** and tokens (see the notebooks for API calls).

### Suggested flow

1. **Cluster setup (admin)**  
   Walk through **Groups** as needed: e.g. `maas-end-user` / `maas-end-user-admin` for visibility into the stack and model deployment; premium/enterprise users for subscription and auth policy demos.

2. **RHOAI dashboard**  
   Show **Subscriptions** and **Auth policies** (or your product’s equivalents).

3. **Free user — Maria**  
   - Sign in as Maria.  
   - Open **Models** and show available models.  
   - In **AI Assets** (or your subscription UI), show **subscription** details for the free tier.  
   - Show **usage metrics**.  
   - Drive traffic until you hit **HTTP 429** (rate limit), consistent with configured token limits.

4. **Notebook (optional; good for APIs)**  
   Run **`BasicDemo.ipynb`** with Jupyter: set `MAAS_BASE` and auth (`OPENSHIFT_TOKEN` or username/password via env), then API key creation, model list, completion, and the 429 loop. If you already have an API key, use **`BasicDemo-no-key.ipynb`** (`MAAS_BASE` and `MAAS_API_KEY` or `API_KEY`).

5. **Deploy a model (UI)**  
   **Models → Deploy model**  
   - **Model URI:** `hf://sshleifer/tiny-gpt2` — the simulator does not need the weights, but the URI must be a real, fetchable model for the pipeline.  
   - **What matters:** `llm-d-inference-sim` uses the **`--model` argument** (see `LLMInferenceServiceConfig` / merged pod args). The samples do not rely on a separate env var for that. **`spec.model.name`** on the `LLMInferenceService` is what MaaS exposes — keep it aligned with `--model`. For another Hugging Face id, override **`spec.template.containers[0].args`** and **`spec.model.name`** on that service, or use your UI if it patches those fields.

## Repository layout

| Path | Purpose |
|------|---------|
| `samples/maas-system/` | Kustomize bundles for free/premium MaaS and simulator models |
| `BasicDemo.ipynb` | Full flow: OpenShift auth, API key, models, completions, 429 |
| `BasicDemo-no-key.ipynb` | Same inference steps with your own `MAAS_API_KEY` / `API_KEY` (no key creation) |
| `ExternalModelsDemo.ipynb` | Gateway chat routes (GPT-4o / Claude presets) |
| `demo-files/` | Reference YAML (not applied automatically) |
| `workaround/` | Optional RBAC samples (e.g. model ref access) |

## Publishing notebooks as static HTML

CI can render the notebooks to **GitHub Pages** without executing cells (no cluster calls in the pipeline). Outputs: **`index.html`** (`BasicDemo`), **`basic-demo-no-key.html`**, **`external-models-demo.html`**. See `.github/workflows/deploy-notebook.yml` and set **Pages → GitHub Actions** in the repository settings.

## Chat completion (curl)

Example **`curl`** for OpenAI-style completions (body includes **`"stream": true`** as in the notebooks). **`curl -N`** turns off curl’s output buffering.

```bash
curl -N -sSk -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d "{\"model\": \"${MODEL_NAME}\", \"prompt\": \"Hello\", \"max_tokens\": 50, \"stream\": true}" \
  "${MODEL_URL}/v1/completions"
```

```bash
stdbuf -oL curl -N -sSk -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d "{\"model\": \"${MODEL_NAME}\", \"prompt\": \"Hello\", \"max_tokens\": 50, \"stream\": true}" \
  "${MODEL_URL}/v1/completions"
```

Set `MODEL_NAME`, `MODEL_URL`, and `API_KEY` the same way as in **`BasicDemo.ipynb`** (or export them after running the setup cells). **`ExternalModelsDemo.ipynb`** prints matching **`curl`** lines after **Load presets**.
