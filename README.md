# 🤖 Generative MCP Avatar Renderer

> A high-quality generative engine that creates talking avatars from a single image and an audio file.
> Built for scalable deployments, this renderer natively integrates via the MCP protocol, enabling seamless, auto-discovered tooling in a distributed system.

---

This project is an AI Talking Head Generator, a tool that uses artificial intelligence to create a video of a person speaking from two simple inputs:

🖼️ A still image of a person's face.

🎤 An audio file of speech.

The AI analyzes both inputs and then generates a new video, animating the person's mouth, lips, and subtle facial expressions to realistically match the words and timing of the voice recording. The result makes it appear as if the person in the static photo is actually speaking.

## 🚀 Features

* **FOMM (head pose)** + **Diff2Lip (diffusion visemes)** with automatic fallback to **SadTalker (+ Wav2Lip)** when VRAM is tight.  
* **MCP STDIO server** *and* FastAPI REST façade live in the same container.  
* CUDA 12.4 + PyTorch 2.3; NVENC H.264 encodes > 200 fps on a V100.  
* Pluggable pipeline (`pipeline.py`) – swap in AnimateDiff, DreamTalk, LIAON‑LipSync, etc.  
* Helm chart & raw manifests request **`nvidia.com/gpu: 1`** and tolerate the **`dedicated=gpu`** taint.  
* KEDA‑ready: ScaledObject samples Kafka lag and spins 0 → N render pods as demand changes.  
* Full CI (CPU‑only) plus Colab notebooks for checkpoint tuning.

---

---

## 🚦 Feature compliance matrix

| Feature                                     | Status  | Notes                                              |
| ------------------------------------------- | ------- | -------------------------------------------------- |
| Realistic facial animation from still image | **✅**   | FOMM / SadTalker for full head + expressions       |
| High‑fidelity lip‑sync                      | **✅**   | Diff2Lip diffusion visemes or Wav2Lip GAN fallback |
| MP4 export (presentation‑ready)             | **✅**   | H.264 NVENC, signed COS/S3 URL                     |
| Live WebRTC streaming                       | ⚠️ Soon | GStreamer NVENC RTP branch in `feature/webrtc`     |
| AI‑agent integration (MCP)                  | **✅**   | STDIO protocol ready, REST remains for demos       |
| Low‑latency incremental synthesis           | ⚠️ R\&D | Needs chunked TTS + sliding‑window Diff2Lip        |

---

## ⚙ Helm deployment (OpenShift)

```bash
helm upgrade --install avatar-renderer charts/avatar-renderer \
  --namespace videogenie --create-namespace \
  --set image.tag=$(git rev-parse --short HEAD)
```

Requests **1 GPU**, 6 GiB RAM, 2 vCPU. The existing VideoGenie KEDA ScaledObject will autoscale pods based on Kafka lag.


# Makefile Guide

```bash
# first‑time developer workflow
make setup
make download-models
make run        # REST server → http://localhost:8080/render

# MCP stdio test
make run-stdio  # then echo '{"tool":"render_avatar", ...}' | ./app/mcp_server.py

# build + run container
make docker-build
make docker-run
```

---
