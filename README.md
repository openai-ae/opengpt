# OpenGPT

OpenGPT is a simple, open-source, and easy-to-use GPT-style text generator. 

**Warning**: This is an idea that I had and I wanted to try it out. 


## Installation

Install the package with pip:

```bash
pip install opengpt
```

## Quickstart

```python
import opengpt

model = opengpt.create_model('facebook/llama-9b', device='cuda', precision='fp16')

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

## Serving Models

You can serve your models with OpenGPT. To do so, you can use the `serve` command:

```bash
opengpt serve facebook/llama-9b --device cuda --precision fp16 --port 5000
```

This will start a server on port 5000. You can then send requests to the server:

```python
import requests

prompt = "The quick brown fox jumps over the lazy dog."

response = requests.post(
    "http://localhost:5000/generate",
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

Note that the server will only accept requests from the same machine. If you want to accept requests from other machines, you can use the `--host` flag to specify the host to bind to.

## Deploying Models

You can also deploy the server to a cloud provider like Jina Cloud or AWS.
To do so, you can use `deploy` command:

- Jina Cloud

```bash
opengpt deploy facebook/llama-9b --device cuda --precision fp16 --provider jina --name opengpt --replicas 2
```

- AWS

```bash
opengpt deploy facebook/llama-9b --device cuda --precision fp16 --provider aws --region us-east-1 --name opengpt --replicas 2
```

This will deploy the model to the cloud provider. You can then send requests to the server:

```python
import requests

prompt = "The quick brown fox jumps over the lazy dog."

response = requests.post(
    "https://opengpt.jina.ai/generate",
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
 
## Advanced Usage

You can also apply the model offloading techniques (based on [FlexTensor](https://github.com/numb3r3/flex-tensor)) to OpenGPT. To do so, you can use the `--offload-percents` flag:

```bash
opengpt serve facebook/llama-9b --device cuda --precision fp16 --port 5000 --offload-percents 10,90,50,50,0,100
```

This will offload parts of the model to the CPU. You can also use the `--offload-strategy` flag to specify the offloading strategy:

```bash
opengpt serve facebook/llama-9b --device cuda --precision fp16 --port 5000 --offload-strategy "cpu,cpu,cpu,cpu,cpu,cpu"
```




## Documentation

For more information, check out the [documentation](https://opengpt.readthedocs.io/en/latest/).