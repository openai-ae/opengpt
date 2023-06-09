# ☄️ OpenGPT

<p align="center">
<a href="https://github.com/jina-ai/opengpt"><img src="https://github.com/jina-ai/opengpt/blob/main/.github/images/logo.png?" alt="OpenGPT: An open-source cloud-native large-scale multimodal model serving framework" width="300px"></a>
<br>
</p>

> "A playful and whimsical vector art of a Stochastic Tigger, wearing a t-shirt with a "GPT" text printed logo, surrounded by colorful geometric shapes.  –ar 1:1 –upbeta"
>
> — Prompts and logo art was produced with  [PromptPerfect](https://promptperfect.jina.ai/) & [Stable Diffusion X](https://clipdrop.co/stable-diffusion)


![](https://img.shields.io/badge/Made%20with-JinaAI-blueviolet?style=flat)
[![PyPI](https://img.shields.io/pypi/v/open_gpt_torch)](https://pypi.org/project/open_gpt_torch/)
[![PyPI - License](https://img.shields.io/pypi/l/open_gpt_torch)](https://pypi.org/project/open_gpt_torch/)

**OpenGPT** is an open-source _cloud-native_ large-scale **_multimodal models_** (LMMs) serving framework. 
It is designed to simplify the deployment and management of large language models, on a distributed cluster of GPUs.
We aim to make it a one-stop solution for a centralized and accessible place to gather techniques for optimizing large-scale multimodal models and make them easy to use for everyone.


## Table of contents

- [Features](#features)
- [Supported models](#supported-models)
- [Get started](#get-started)
- [Build a model serving in one line](#build-a-model-serving-in-one-line)
- [Cloud-native deployment](#cloud-native-deployment)
- [Roadmap](#roadmap)

## Features

OpenGPT provides the following features to make it easy to deploy and serve **large multi-modal models** (LMMs) at scale:

- Support for multi-modal models on top of large language models
- Scalable architecture for handling high traffic loads
- Optimized for low-latency inference
- Automatic model partitioning and distribution across multiple GPUs
- Centralized model management and monitoring
- REST API for easy integration with existing applications

## Updates

- **2023-05-12**: 🎉We have released the first version `v0.0.1` of OpenGPT. You can install it with `pip install open_gpt_torch`.

## Supported Models

<details>

OpenGPT supports the following models out of the box:

- LLM (Large Language Model)

  - [LLaMA](https://ai.facebook.com/blog/large-language-model-llama-meta-ai/): open and efficient foundation language models by Meta
  - [Pythia](https://github.com/EleutherAI/pythia): a collection of models developed to facilitate interpretability research by EleutherAI
  - [StableLM](https://github.com/Stability-AI/StableLM): series of large language models by Stability AI
  - [Vicuna](https://vicuna.lmsys.org/): a chat assistant fine-tuned from LLaMA on user-shared conversations by LMSYS
  - [MOSS](https://txsun1997.github.io/blogs/moss.html): conversational language model from Fudan University

- LMM (Large Multi-modal Model)

  - [OpenFlamingo](https://github.com/mlfoundations/open_flamingo): an open source version of DeepMind's [Flamingo](https://www.deepmind.com/blog/tackling-multiple-tasks-with-a-single-visual-language-model) model
  - [MiniGPT-4](https://minigpt-4.github.io/): aligns a frozen visual encoder with a frozen LLM, Vicuna, using just one projection layer. 

For more details about the supported models, please see the [Model Zoo](./MODEL_ZOO.md).

</details>


## Roadmap

You can view our roadmap with features that are planned, started, and completed on the [Roadmap discussion](https://github.com/jina-ai/opengpt/discussions/categories/roadmap) category.

## Get Started

### Installation

Install the package with `pip`:

```bash
pip install open_gpt_torch
```

### Quickstart

```python
import open_gpt

model = open_gpt.create_model(
    'stabilityai/stablelm-tuned-alpha-3b', device='cuda', precision='fp16'
)

prompt = "The quick brown fox jumps over the lazy dog."

output = model.generate(
    prompt,
    max_length=100,
    temperature=0.9,
    top_k=50,
    top_p=0.95,
    repetition_penalty=1.2,
    do_sample=True,
    num_return_sequences=1,
)
```

We use the [stabilityai/stablelm-tuned-alpha-3b](https://huggingface.co/stabilityai/stablelm-tuned-alpha-3b) as the open example model as it is relatively small and fast to download.

> **Warning**
> In the above example, we use `precision='fp16'` to reduce the memory usage and speed up the inference with some loss in accuracy on text generation tasks. 
> You can also use `precision='fp32'` instead as you like for better performance. 

> **Note**
> It usually takes a while (several minutes) for the first time to download and load the model into the memory.


In most cases of large model serving, the model cannot fit into a single GPU. To solve this problem, we also provide a `device_map` option (supported by `accecleate` package) to automatically partition the model and distribute it across multiple GPUs:

```python
model = open_gpt.create_model(
    'stabilityai/stablelm-tuned-alpha-3b', precision='fp16', device_map='balanced'
)
```

In the above example, `device_map="balanced"` evenly split the model on all available GPUs, making it possible for you to serve large models.

> **Note**
> The `device_map` option is supported by the [accelerate](https://github.com/huggingface/accelerate) package. 


See [examples on how to use opengpt with different models.](./examples) 🔥


## Build a model serving in one line

To do so, you can use the `serve` command:

```bash
opengpt serve yahma/llama-7b-hf --precision fp16 --device_map balanced
```

or 

```bash
opengpt serve yahma/llama-7b-hf --adapter jinaai/alpaca-lora --precision bit8 --device_map balanced
```

if you want to use adapter to optimize VRAM usage.

💡 **Tip**: you can inspect the available options with `opengpt serve --help`.

> **Note**
> `adapter` is only available for `LLaMA` model for now.

This will start a gRPC and HTTP server listening on port `51000` and `52000` respectively. 
Once the server is ready, as shown below:
<details>
<summary>Click to expand</summary>
<img src="https://github.com/jina-ai/opengpt/blob/main/.github/images/serve_ready.png" width="600px">
</details>

You can then send requests to the server:

```python
import requests

prompt = "The quick brown fox jumps over the lazy dog."

response = requests.post(
    "http://localhost:51000/generate",
    json={
        "prompt": prompt,
        "max_length": 100,
        "temperature": 0.9,
        "top_k": 50,
        "top_p": 0.95,
        "repetition_penalty": 1.2,
        "do_sample": True,
        "num_return_sequences": 1,
    },
)
```

What's more, we also provide a [Python client](https://github.com/jina-ai/inference-client/) (`inference-client`) for you to easily interact with the server:

```python
from open_gpt import Client

client = Client()

# connect to the model server
model = client.get_model(endpoint='grpc://0.0.0.0:51000')

prompt = "The quick brown fox jumps over the lazy dog."

output = model.generate(
    prompt,
    max_length=100,
    temperature=0.9,
    top_k=50,
    top_p=0.95,
    repetition_penalty=1.2,
    do_sample=True,
    num_return_sequences=1,
)
```

💡 **Tip**: To display the list of available commands, please use the `list` command.

## Cloud-native deployment

You can also deploy the server to a cloud provider like Jina Cloud or AWS.
To do so, you can use `deploy` command:

### Jina Cloud

using predefined executor

```bash
opengpt deploy yahma/llama-7b-hf --adapter jinaai/alpaca-lora --precision fp16 --cloud jina --replicas 2
```

or using customized YAML file

```bash
opengpt deploy --config flow.yml
```

It will give you a HTTP url and a gRPC url by default:
```bash
https://activate-puma-3defr4e32-http.wolf.jina.ai
grpcs://activate-puma-3defr4e32-grpc.wolf.jina.ai
```

You can send request using:

```python
import requests

prompt = "The quick brown fox jumps over the lazy dog."

response = requests.post(
    "http://activate-puma-3defr4e32-http.wolf.jina.ai/generate",
    json={
        "prompt": prompt,
        "max_length": 100,
        "temperature": 0.9,
        "top_k": 50,
        "top_p": 0.95,
        "repetition_penalty": 1.2,
        "do_sample": True,
        "num_return_sequences": 1,
    },
)
```

or using `inference-client`:
```python
from open_gpt import Client

client = Client()

# using grpc protocol
model = client.get_model(endpoint='grpcs://activate-puma-3defr4e32-grpc.wolf.jina.ai')

prompt = "The quick brown fox jumps over the lazy dog."

output = model.generate(
    prompt,
    max_length=100,
    temperature=0.9,
    top_k=50,
    top_p=0.95,
    repetition_penalty=1.2,
    do_sample=True,
    num_return_sequences=1,
)
```

Parameters that can be configured for `opengpt deploy`:
- `cloud`: the cloud provider to use, e.g. `jina` for Jina Cloud, `aws` for AWS.
- `adapter`: (Optional) the adapter to use, e.g. `jinaai/alpaca-lora` denotes LoRA weights used for llama-7b. Can be a string 
denoting the name id which is published on Huggingface, or a path of a directory which contains adapter weights and config file.
Default is `None`.
- `device_map`: (Optional) the device map to use, e.g. `balanced` for evenly splitting the model on all available GPUs. Default is `balanced`.
- `replicas`: (Optional) the number of replicas to deploy. Default is 1.
- `precision`: (Optional) the precision to use, can be `fp16` or `bit8` or `bit4`. Default is `fp16`.
- `config`: (Optional) the Jina Flow YAML file to use. If specified, all other parameters (including `model_name`) will be **ignored**. This is especially 
useful when you want to customize the Flow YAML file. Default is `None`.

The template for the Flow YAML file is as follows:

```yaml
jtype: Flow
jcloud:
  version: 3.14.1
with:
  monitoring: true
  name: my_opengpt_flow
  prefetch: 1
  timeout_ready: -1
  env:
    JINA_LOG_LEVEL: DEBUG
gateway:
  port:
    - 52000
    - 51000
  protocol:
    - http
    - grpc
executors:
  - name: llm_executor
    uses:
      jtype: CausualLMExecutor # your executor type, which is the name of model class
      py_modules:
        - __init__.py # path to the executor's __init__.py file
    uses_with:
      model_name_or_path: yahma/llama-7b-hf # your model name or model path
      adapter_name_or_path: jinaai/alpaca-lora # your adapter name or path
      device_map: balanced # device map
      precision: bit8 # precision
    timeout_ready: 3600000
    jcloud:
      resources:
        instance: G3 # set you instance type here
```

- Flow related config
```yaml
with:
  monitoring: true
  name: my_opengpt_flow
  prefetch: 1
  timeout_ready: -1
  env:
    JINA_LOG_LEVEL: DEBUG
```

| Parameters |                                                                                                      Description                                                                                                     |
|------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|
| prefetch   | Control the maximum streamed request inside the Flow at any given time, default is None , which means no limit. Setting prefetch a small number helps solve the OOM problem, but may  slow down the streaming a bit. |
- Gateway related config

```yaml
gateway:
  port:
    - 52000
    - 51000
  protocol:
    - http
    - grpc
```

| Parameters | Description                                                                          |
|------------|--------------------------------------------------------------------------------------|
| protocol   | The communication protocol between server and client. Can be grpc , http, websocket. |
| port       | The port which is used for communication between server and client.                  |

- Executor related config

```yaml
executors:
  - name: llm_executor
    uses:
      jtype: CausualLMExecutor # your executor type, which is the name of model class
      py_modules:
        - __init__.py # path to the executor's __init__.py file
    uses_with:
      model_name_or_path: yahma/llama-7b-hf # your model name or model path
      adapter_name_or_path: jinaai/alpaca-lora # your adapter name or path
      device_map: balanced # device map
      precision: bit8 # precision
    timeout_ready: 3600000
    jcloud:
      resources:
        instance: G3 # set you instance type here
```

| Parameters           | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
|----------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| jtype                | Name of the Executor model class.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| py_modules           | List of strings defining the Executor’s Python dependencies. Most notably this must include the Python file containing the Executor definition itself, as well as any other files it imports.                                                                                                                                                                                                                                                                                                                                                           |
| model_name_or_path   | The name of the model to use. Can be either:   - A string, the *model id* of a predefined tokenizer hosted inside a model repo on huggingface.co.   Valid model ids can be located at the root-level, like `bert-base-uncased`, or namespaced under a   user or organization name, like `dbmdz/bert-base-german-cased`.   - A path to a *directory* containing vocabulary files required by the tokenizer, for instance saved   using the [`~tokenization_utils_base.PreTrainedTokenizerBase.save_pretrained`] method, e.g.,   `./my_model_directory/`. |
| adapter_name_or_path | The name of the adapter configuration to use. Can be either:   - A string, the `model id` of a Lora configuration hosted inside a model repo on the Hugging Face Hub.   - A path to a directory containing a Lora configuration file saved using the `save_pretrained` method (`./my_lora_config_directory/`).                                                                                                                                                                                                                                          |
| device_map           | Define how the model lies across several devices.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| precision            | The precision which is used for inference. Can be fp16 / bit8 / bit4. You need to set `precision=bit8` if you want to use LoRA or `precision=bit4` if QLoRA is used                                                                                                                                                                                                                                                                                                                                                                                     |
| instance             | GPU instance types on JCloud. For more information about GPU instance type, click [here](https://docs.jina.ai/concepts/jcloud/configuration/#gpu-tiers)                                                                                                                                                                                                                                                                                                                                                                                                 |

> **Note**
> - `uses_with` may be change if customized executor is used. It's a key-value map that defines the arguments of the 
> executor’s `__init__` method. See more details [here](https://docs.jina.ai/concepts/orchestration/add-executors/).


### AWS

TBD

## Contributing

We welcome contributions from the community! To contribute, please submit a pull request following our contributing guidelines.

## License

OpenGPT is licensed under the Apache License, Version 2.0. See LICENSE for the full license text.