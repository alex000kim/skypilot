<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/skypilot-org/skypilot/master/docs/source/images/skypilot-wide-dark-1k.png">
    <img alt="SkyPilot" src="https://raw.githubusercontent.com/skypilot-org/skypilot/master/docs/source/images/skypilot-wide-light-1k.png" width=55%>
  </picture>
</p>

<p align="center">
  <a href="https://docs.skypilot.co/">
    <img alt="Documentation" src="https://img.shields.io/badge/docs-gray?logo=readthedocs&logoColor=f5f5f5">
  </a>

  <a href="https://github.com/skypilot-org/skypilot/releases">
    <img alt="GitHub Release" src="https://img.shields.io/github/release/skypilot-org/skypilot.svg">
  </a>

  <a href="http://slack.skypilot.co">
    <img alt="Join Slack" src="https://img.shields.io/badge/SkyPilot-Join%20Slack-blue?logo=slack">
  </a>

  <a href="https://github.com/skypilot-org/skypilot/releases">
    <img alt="Downloads" src="https://img.shields.io/pypi/dm/skypilot">
  </a>

</p>

<h3 align="center">
    Run AI on Any Infrastructure
</h3>

<div align="center">

#### [🌟 **SkyPilot Demo** 🌟: Click to see a 1-minute tour](https://demo.skypilot.co/dashboard/)

</div>


SkyPilot is a system to run, manage, and scale AI workloads on any AI infrastructure.

SkyPilot gives **AI teams** a simple interface to run jobs on any infra.
**Infra teams** get a unified control plane to manage any AI compute — with advanced scheduling, scaling, and orchestration.

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="./docs/source/images/skypilot-abstractions-long-2-dark.png">
  <img src="./docs/source/images/skypilot-abstractions-long-2.png" alt="SkyPilot Abstractions">
</picture>

-----

:fire: *News* :fire:
- [Dec 2025] **SkyPilot v0.11** released: Multi-Cloud Pools, Fast Managed Jobs, Enterprise-Readiness at Large Scale, Programmability. [**Release notes**](https://github.com/skypilot-org/skypilot/releases/tag/v0.11.0)
- [Dec 2025] **SkyPilot Pools** released: Run batch inference and other jobs on a managed pool of warm workers (across clouds or clusters). [**blog**](https://blog.skypilot.co/skypilot-pools-deepseek-ocr/), [**docs**](https://docs.skypilot.co/en/latest/examples/pools.html)
- [Dec 2025] Train **an agent to use Google Search** as a tool with RL on your Kubernetes or clouds: [**blog**](https://blog.skypilot.co/verl-tool-calling/), [**example**](./llm/verl/)

## Overview

- **Simple**: Spin up compute, manage jobs, and auto-recover — with a unified YAML/Python API
- **Kubernetes-native**: Slurm-like UX, gang scheduling, multi-cluster, local dev experience on K8s
- **Multi-cloud**: One interface across reserved GPUs, Kubernetes, Slurm, and 20+ clouds, with [auto-failover](https://docs.skypilot.co/en/latest/examples/auto-failover.html)
- **Cost-efficient**: Autostop, [spot instances](https://docs.skypilot.co/en/latest/examples/managed-jobs.html#running-on-spot-instances) (3-6x savings), and intelligent scheduling
- **No code changes**: Drop in your existing GPU, TPU, and CPU workloads

Install with pip:
```bash
# Choose your clouds:
pip install -U "skypilot[kubernetes,aws,gcp,azure,oci,nebius,lambda,runpod,fluidstack,paperspace,cudo,ibm,scp,seeweb,shadeform]"
```
To get the latest features and fixes, use the nightly build or [install from source](https://docs.skypilot.co/en/latest/getting-started/installation.html):
```bash
# Choose your clouds:
pip install "skypilot-nightly[kubernetes,aws,gcp,azure,oci,nebius,lambda,runpod,fluidstack,paperspace,cudo,ibm,scp,seeweb,shadeform]"
```

<p align="center">
  <img src="docs/source/_static/intro.gif" alt="SkyPilot">
</p>

Current supported infra: Kubernetes, Slurm, AWS, GCP, Azure, OCI, CoreWeave, Nebius, Lambda Cloud, RunPod, Fluidstack,
Cudo, Digital Ocean, Paperspace, Cloudflare, Samsung, IBM, Vast.ai, VMware vSphere, Seeweb, Prime Intellect, Shadeform.
<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/skypilot-org/skypilot/master/docs/source/images/cloud-logos-dark.png">
    <img alt="SkyPilot" src="https://raw.githubusercontent.com/skypilot-org/skypilot/master/docs/source/images/cloud-logos-light.png" width=85%>
  </picture>
</p>
<!-- source xcf file: https://drive.google.com/drive/folders/1S_acjRsAD3T14qMeEnf6FFrIwHu_Gs_f?usp=drive_link -->


## Getting started
You can find our documentation [here](https://docs.skypilot.co/).
- [Installation](https://docs.skypilot.co/en/latest/getting-started/installation.html)
- [Quickstart](https://docs.skypilot.co/en/latest/getting-started/quickstart.html)
- [CLI reference](https://docs.skypilot.co/en/latest/reference/cli.html)

## SkyPilot in 1 minute

A SkyPilot task specifies: resource requirements, data to be synced, setup commands, and the task commands.

Once written in this [**unified interface**](https://docs.skypilot.co/en/latest/reference/yaml-spec.html) (YAML or Python API), the task can be launched on any available infra (Kubernetes, Slurm, cloud, etc.).  This avoids vendor lock-in, and allows easily moving jobs to a different provider.

Paste the following into a file `my_task.yaml`:

```yaml
resources:
  accelerators: A100:8

num_nodes: 1

workdir: ~/torch_examples

setup: |
  cd mnist
  pip install -r requirements.txt

run: |
  cd mnist
  python main.py --epochs 1
```

Prepare the workdir by cloning:
```bash
git clone https://github.com/pytorch/examples.git ~/torch_examples
```

Launch with `sky launch` (note: [access to GPU instances](https://docs.skypilot.co/en/latest/cloud-setup/quota.html) is needed for this example):
```bash
sky launch my_task.yaml
```

SkyPilot finds the cheapest available infra, provisions GPUs, syncs your workdir, runs setup, and streams logs. See [Quickstart](https://docs.skypilot.co/en/latest/getting-started/quickstart.html) to get started.

## Runnable examples

See [**SkyPilot examples**](https://docs.skypilot.co/en/docs-examples/examples/index.html) that cover: development, training, serving, LLM models, AI apps, and common frameworks.

Latest featured examples:

| Task | Examples |
|----------|----------|
| Training | [Verl](https://docs.skypilot.co/en/latest/examples/training/verl.html), [Finetune Llama 4](https://docs.skypilot.co/en/latest/examples/training/llama-4-finetuning.html), [TorchTitan](https://docs.skypilot.co/en/latest/examples/training/torchtitan.html), [PyTorch](https://docs.skypilot.co/en/latest/getting-started/tutorial.html), [DeepSpeed](https://docs.skypilot.co/en/latest/examples/training/deepspeed.html), [NeMo](https://docs.skypilot.co/en/latest/examples/training/nemo.html), [Ray](https://docs.skypilot.co/en/latest/examples/training/ray.html), [Unsloth](https://docs.skypilot.co/en/latest/examples/training/unsloth.html), [Jax/TPU](https://docs.skypilot.co/en/latest/examples/training/tpu.html) |
| Serving | [vLLM](https://docs.skypilot.co/en/latest/examples/serving/vllm.html), [SGLang](https://docs.skypilot.co/en/latest/examples/serving/sglang.html), [Ollama](https://docs.skypilot.co/en/latest/examples/serving/ollama.html) |
| Models | [DeepSeek-R1](https://docs.skypilot.co/en/latest/examples/models/deepseek-r1.html), [Llama 4](https://docs.skypilot.co/en/latest/examples/models/llama-4.html), [Llama 3](https://docs.skypilot.co/en/latest/examples/models/llama-3.html), [CodeLlama](https://docs.skypilot.co/en/latest/examples/models/codellama.html), [Qwen](https://docs.skypilot.co/en/latest/examples/models/qwen.html), [Kimi-K2](https://docs.skypilot.co/en/latest/examples/models/kimi-k2.html), [Kimi-K2-Thinking](https://docs.skypilot.co/en/latest/examples/models/kimi-k2-thinking.html), [Mixtral](https://docs.skypilot.co/en/latest/examples/models/mixtral.html) |
| AI apps | [RAG](https://docs.skypilot.co/en/latest/examples/applications/rag.html), [vector databases](https://docs.skypilot.co/en/latest/examples/applications/vector_database.html) (ChromaDB, CLIP) |
| Common frameworks | [Airflow](https://docs.skypilot.co/en/latest/examples/frameworks/airflow.html), [Jupyter](https://docs.skypilot.co/en/latest/examples/frameworks/jupyter.html), [marimo](https://docs.skypilot.co/en/latest/examples/frameworks/marimo.html)  |

Source files can be found in [`llm/`](https://github.com/skypilot-org/skypilot/tree/master/llm) and [`examples/`](https://github.com/skypilot-org/skypilot/tree/master/examples).

## More information

- [Docs](https://docs.skypilot.co/en/latest/) · [Blog](https://blog.skypilot.co/) · [Case Studies](https://blog.skypilot.co/case-studies/) · [Community](https://blog.skypilot.co/community/)
- Follow: [Slack](http://slack.skypilot.co) · [X](https://twitter.com/skypilot_org) · [LinkedIn](https://www.linkedin.com/company/skypilot-oss/)
- Research: [SkyPilot (NSDI '23)](https://www.usenix.org/system/files/nsdi23-yang-zongheng.pdf) · [SkyServe (EuroSys '25)](https://arxiv.org/pdf/2411.01438) · [Spot policy (NSDI '24)](https://www.usenix.org/conference/nsdi24/presentation/wu-zhanghao) · [Sky Computing](https://arxiv.org/abs/2205.07147)

SkyPilot was started at the [Sky Computing Lab](https://sky.cs.berkeley.edu) at UC Berkeley. See [Concept: Sky Computing](https://docs.skypilot.co/en/latest/sky-computing.html) for the project's origin and vision.

## Questions and feedback
We are excited to hear your feedback:
* For issues and feature requests, please [open a GitHub issue](https://github.com/skypilot-org/skypilot/issues/new).
* For questions, please use [GitHub Discussions](https://github.com/skypilot-org/skypilot/discussions).

For general discussions, join us on the [SkyPilot Slack](http://slack.skypilot.co).

## Contributing
We welcome all contributions to the project! See [CONTRIBUTING](CONTRIBUTING.md) for how to get involved.
