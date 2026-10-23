# Troubleshooting Reference
## Manufacturing Defect Inspection Lab — Student Reference

> Try asking Bob first: describe the error you see and Bob will diagnose it. Use this file when you need detailed fix steps.
> All fixes below reference `studentXX` — substitute your own student ID.

---

## T1 — Can't open the workbench (URL not loading)

**Cause:** Workbench pod may be starting up.
**Fix:** Wait 2 minutes. If still not working, go to RHOAI Dashboard → Data Science Projects → `studentXX` → Workbenches → click ⋮ → **Start**.

> 🤖 **Bob prompt:**
> ```
> In namespace studentXX, check if the workbench pod my-workbench-0 is running and
> show me any recent error events.
> ```

---

## T2 — `ModuleNotFoundError: No module named 'ultralytics'` (or `onnxruntime`)

**Symptom:** Notebook 1 Check 1 fails with `ModuleNotFoundError` on `ultralytics` or `onnxruntime`.

**Cause:** The RHOAI PyTorch workbench image does not pre-install these packages. Cell 0a in every notebook installs them automatically — but only if you **run cells in order from the top**.

**Fix — if you skipped Cell 0a or ran cells out of order:**
1. In JupyterLab: **Kernel → Restart Kernel and Run All Cells → Restart**
2. Wait ~60 seconds for Cell 0a to complete the install
3. All subsequent cells will find the packages installed

**Fix — if Cell 0a itself failed** (e.g. network timeout):
```python
# Run this in a new notebook cell:
import subprocess, sys
subprocess.run([sys.executable, '-m', 'pip', 'install',
                'ultralytics==8.4.96', 'onnxruntime==1.25.0'], check=True)
```

> ℹ️ The install writes to `~/.local` (user site-packages). It persists for the lifetime of the workbench pod. If the workbench is restarted, Cell 0a will reinstall automatically on the next run (~60 s).

---

## T3 — Git clone fails

**Symptom:** `fatal: ...` error during `git clone`.

**Fix:**
```bash
# Check network from the workbench terminal
curl -I https://github.com 2>&1 | head -3
# If reachable, retry:
rm -rf ~/lab-materials
git clone https://github.com/faheemshai/1512_model_training.git ~/lab-materials
```

---

## T4 — S3 not reachable (Notebook 1, Check 2 fails)

**Symptom:** `❌ S3 not reachable` in Notebook 1.

**Cause A:** Data connection not attached to workbench.
**Fix:** RHOAI Dashboard → `studentXX` → Workbenches → `my-workbench` → Edit → ensure `studentXX-data-connection` is selected → Save → restart workbench.

**Cause B:** Env vars not set. Check with:
```python
import os
for k in ['AWS_S3_ENDPOINT','AWS_ACCESS_KEY_ID','AWS_SECRET_ACCESS_KEY','AWS_S3_BUCKET']:
    print(k, '=', os.environ.get(k,'NOT SET'))
```
All `NOT SET` → the workbench isn't connected to the data connection. Follow Cause A fix.

---

## T5 — Training fails (memory error or kernel crash)

**Symptom:** `RuntimeError: out of memory` or kernel restarts during Notebook 3.

**Fix:** Edit the Configuration cell at the top of Notebook 3:
```python
EPOCHS           = 10   # reduce from 30
BATCH_SIZE       = 8    # reduce from 16
IMAGES_PER_CLASS = 40   # reduce from 60
```
Then re-run all cells from the top.

---

## T6 — InferenceService stuck `READY=False`

**Symptom:** `oc get inferenceservice defect-classifier -n studentXX` shows `READY=False` after 5+ minutes.

> 🤖 **Bob prompt:**
> ```
> In namespace studentXX, the InferenceService defect-classifier is not becoming ready.
> Check the predictor pod: oc get pods -n studentXX | grep defect-classifier
> Show me the pod's events and init container logs to diagnose the failure.
> ```

**Cause A:** Wrong container name in ServingRuntime
```
failed to find kserve-container in ServingRuntime containers
```
→ ServingRuntime container must be named `kserve-container` (not `ovms`). Recreate with correct name.

**Cause B:** Wrong image tag (`manifest unknown`)
```
ErrImagePull: manifest unknown for quay.io/modh/openvino_model_server:2026.1.0-gpu
```
→ Use `quay.io/modh/openvino_model_server:rhoai-2.19`. Recreate ServingRuntime with correct tag.

**Cause C:** LimitRange violation (pod rejected at admission)
```
pods forbidden: maximum memory usage per Container is Xgi, but limit is 24Gi
```
→ Contact instructor. LimitRange Container max must be 24Gi.

**Cause D:** Wrong S3 path
```
No version found for model in path: /mnt/models
```
→ The path has a `/1/` suffix. Ask Bob:
> 🤖 **Bob prompt:**
> ```
> In namespace studentXX, update the InferenceService defect-classifier storage path.
> Remove the /1/ suffix — change path to models/defect-classifier/ (not models/defect-classifier/1/)
> ```

**Cause E:** S3 credentials wrong (`AccessDenied` in logs)
> 🤖 **Bob prompt:**
> ```
> In namespace studentXX, check if the secret studentXX-data-connection exists and
> has the correct keys: AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, AWS_S3_BUCKET,
> AWS_S3_ENDPOINT, AWS_DEFAULT_REGION
> ```

---

## T7 — Inference returns HTTP 503

**Symptom:** Notebook 4 gets `HTTP 503` from the inference endpoint.

**Cause:** The KServe predictor's Kubernetes service is headless (`clusterIP: None`) — OpenShift Routes cannot route to headless services.

> 🤖 **Bob prompt:**
> ```
> In namespace studentXX, the inference endpoint returns 503. Check if the service
> defect-classifier-predictor is headless (clusterIP: None). If so:
> 1. Create a ClusterIP service named defect-classifier-ext on port 8888 with
>    selector app=isvc.defect-classifier-predictor
> 2. Create a Route named defect-classifier-ext pointing to defect-classifier-ext
>    with TLS edge termination on port 8888
>    (name it defect-classifier-ext NOT defect-classifier — KServe deletes that name)
> ```

---

## T8 — Web app: PostgreSQL ImagePullBackOff

**Symptom:** PostgreSQL pod stuck in `ImagePullBackOff`. Events show `lookup registry.access.redhat.io: no such host`.

**Cause:** `registry.access.redhat.io` DNS resolution fails on this cluster. Must use `registry.redhat.io` with the SHA-pinned image.

**Fix:** Step 10a Bob prompt uses the correct `registry.redhat.io` SHA-pinned image. If the deployment was created manually with the wrong registry, delete and re-run Step 10a.

> 🤖 **Bob prompt:**
> ```
> In namespace studentXX, delete the postgresql deployment and re-deploy it using
> image registry.redhat.io/rhel9/postgresql-15@sha256:8b8b95606e6d06eb866c825ddcc758223a3c34424477c43ce8e3b2b82da62054
> ```

---

## T9 — Web app: Backend CrashLoopBackOff

**Symptom:** Backend pod keeps restarting.

> 🤖 **Bob prompt:**
> ```
> In namespace studentXX, the backend pod is in CrashLoopBackOff.
> Show me the last 30 lines of the backend pod logs to diagnose the failure.
> ```

**Common causes and fixes:**

| Symptom in logs | Cause | Fix |
|----------------|-------|-----|
| `Permission denied: '/.local'` | Container running without `HOME=/tmp` | Re-run Step 10b Bob prompt — manifest sets `HOME=/tmp` correctly |
| `ModuleNotFoundError: python-multipart` | Package missing from pip install | Re-run Step 10b Bob prompt — manifest includes `python-multipart` |
| `INFERENCE_URL not set` | backend-config ConfigMap missing INFERENCE_URL | Ask Bob to patch: `oc patch configmap backend-config -n studentXX -p '{"data":{"INFERENCE_URL":"https://defect-classifier-ext-studentXX.{domain}"}}'` then `oc rollout restart deployment/backend -n studentXX` |

---

## T10 — Web app: Frontend CrashLoopBackOff

**Symptom:** Frontend pod crashes. Logs show `Permission denied` for `cp`, `sed`, or `mkdir`.

**Cause:** nginx:alpine has a read-only container filesystem. The manifest must use an `initContainer` to write the nginx config into an `emptyDir` volume instead of trying to write into the image layer.

**Fix:** Re-run Step 10c Bob prompt — it creates the deployment with the correct `initContainer` + `emptyDir` pattern.

---

## T11 — Web app: `Unexpected token '<', "<html>..."` error when classifying

**Symptom:** Clicking Inspect Component shows `Error: Unexpected token '<', "<html><h"... is not valid JSON`.

**Cause:** The frontend `BACKEND` constant in `index.html` still contains the placeholder `__BACKEND_URL__` — the backend URL was not injected during deploy.

**Fix:** Re-run Step 10c Bob prompt, providing the correct backend URL:
```
https://backend-studentXX.{domain}
```

---

## T12 — Web app: Backend times out becoming ready

**Symptom:** Step 10d Bob prompt reports backend deployment not yet ready.

**Cause:** Backend installs Python packages at first startup (~90–120 s). On a loaded cluster this can be slower.

**Fix:** Wait 2 minutes and re-run the Step 10d verification prompt. If still not ready after 5 minutes:
> 🤖 **Bob prompt:**
> ```
> In namespace studentXX, show me the last 20 lines of backend pod logs.
> Is it still installing packages or has it crashed?
> ```

---

## T13 — App shows correct class but wrong verdict (all showing PASS)

**Symptom:** App classifies correctly (e.g. predicts `crack`) but every result shows a PASS verdict.

**Cause:** The backend ConfigMap `CLASSES` order does not match the ONNX model's alphabetical output order. The correct order is `contamination,crack,pass,scratch`.

**Fix:**
> 🤖 **Bob prompt:**
> ```
> In namespace studentXX, patch the backend-config ConfigMap to set
> CLASSES=contamination,crack,pass,scratch and restart the backend deployment.
> ```

Or manually:
```bash
oc patch configmap backend-config -n studentXX \
  -p '{"data":{"CLASSES":"contamination,crack,pass,scratch","MODEL_NAME":"defect-classifier","IMG_SIZE":"96"}}'
oc rollout restart deployment/backend -n studentXX
```

---

## T14 — DSPA not becoming ready

**Symptom:** `oc get dspa dspa -n studentXX` shows `READY=False` after 5+ minutes.

**Cause A:** Wrong CRD apiVersion (`v1alpha1` instead of `v1`).
**Fix:** Delete and recreate using `datasciencepipelinesapplications.opendatahub.io/v1`.

**Cause B:** NooBaa not ready.
**Fix:**
```bash
oc get noobaa -n openshift-storage
# Must show Ready before creating OBCs or DSPAs
```

---

## T15 — OBC stuck in Pending

**Symptom:** `oc get obc studentXX-bucket -n studentXX` shows `STATUS=Pending` indefinitely.

**Cause:** NooBaa operator not fully initialised.
**Fix:**
```bash
oc get noobaa -n openshift-storage
# Wait until PHASE=Ready, then retry OBC creation
```

---

*Troubleshooting Reference — Manufacturing Defect Inspection Lab — RHOAI 3.4.3*
