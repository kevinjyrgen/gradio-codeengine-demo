# Gradio App on IBM Code Engine

A minimal [Gradio](https://www.gradio.app/) web app, containerized with Docker and deployed serverlessly to **IBM Cloud Code Engine** with a public URL.

Built while working through an IBM Skills Network guided project — the focus is the full deployment pipeline (local app → container image → serverless deploy), not the app itself.

---

## What it does

A tiny "greet" interface: enter a name, set an intensity with a slider, and the app returns `Hello, <name>!` with the exclamation mark repeated `intensity` times. It's deliberately trivial so the interesting part is everything around it — the container build and the cloud deployment.

---

## Files

| File | Purpose |
|---|---|
| `demo.py` | The Gradio app. Binds to `0.0.0.0:7860` so it's reachable inside a container. |
| `requirements.txt` | Python dependencies (`gradio`, `requests`). |
| `Dockerfile` | Build recipe — `python:3.10` base, installs deps, runs the app. |

---

## Run locally

```bash
python3 -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate

pip install -r requirements.txt
python3 demo.py
```

Open `http://0.0.0.0:7860` in your browser.

---

## Run in a container

```bash
docker build -t gradio-demo .
docker run -p 7860:7860 gradio-demo
```

Then visit `http://localhost:7860`.

---

## Deploy to IBM Code Engine

The image was built and deployed with the IBM Cloud CLI. The key steps:

```bash
# 1. Build the container image from local source and push to IBM Container Registry
ibmcloud ce build create --name build-local-dockerfile1 \
    --build-type local --size large \
    --image us.icr.io/${SN_ICR_NAMESPACE}/myapp1 \
    --registry-secret icr-secret

ibmcloud ce buildrun submit --name buildrun-local-dockerfile1 \
    --build build-local-dockerfile1 --source .

# 2. Deploy the image as a serverless app
ibmcloud ce application create --name demo1 \
    --image us.icr.io/${SN_ICR_NAMESPACE}/myapp1 \
    --registry-secret icr-secret --es 2G \
    --port 7860 --minscale 1

# 3. Get the public URL
ibmcloud ce app get --name demo1 --output url
```

**Two things that have to stay consistent across the pipeline:** the port (`7860`, in both `demo.py` and the deploy `--port`), and the image name (referenced in both the build and the deploy). `--minscale 1` keeps one instance warm so there's no cold start on first request.

---

## Tech stack

Gradio · Docker · IBM Cloud Code Engine · IBM Container Registry

---

## License

Licensed under the [Apache License 2.0](LICENSE), matching the IBM Skills Network guided project this was based on.
