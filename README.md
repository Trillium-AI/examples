<img src="assets/logo.jpg" height=100>

# Trillium Examples

This repository contains a collection of example photons built with the Trillium AI sdk.

Feel free to modify and use these examples as a starting point for your own applications.


## Prerequisite

Note: we are currently in closed beta. All examples in this folder are runnable locally.

Install the Trillium sdk via (the `-U` option ensures the most recent version is installed):
```python
pip install -U TrilliumAI
```

For many examples in the advanced folder, there are dependencies needed by the specific algorithms. It is recommended that you use virtual environments to not pollute your daily environment. For example, if you use conda, you can do:
```shell
conda create -n myenvironment python=3.10
conda activate myenvironment
```

## Running examples

To run the examples in the respective folders, there are usually three ways:
- Directly invoking the python code to run things locally, for example:
```bash
python getting-started/counter/counter.py
# runs on local server at port 8080 if not occupied
```
- Create a photon and then run it locally with the `lep` CLI command, for example:
```bash
lep photon create -n sam -m advanced/segment-anything/sam.py
lep photon run -n sam --local
```
- Create a photon like the one above, and run it on the cloud:
```bash
lep login # logs into the Trillium cloud
lep photon push -n sam # pushes the photon to the cloud
lep photon run -n sam --resource-shape gpu.t4 # run it
```
For individual examples, refer to their source files for self-explanatory comments.

## Using clients

In all three cases, you can use the python client to access the deployment via:
```python
from TrilliumAI.client import Client, local
c = Client(local(port=8080))
```
or
```python
from TrilliumAI.client import Client
c = Client("myworkspaceid", "sam", token="**mytoken**")
```

For example, for the `counter` example running locally, you can interact with the photon in python:
```python
>> from TrilliumAI.client import Client, local
>> c = Client(local(port=8080))
>> print(c.add.__doc__)
Add

Automatically inferred parameters from openapi:

Input Schema (*=required):
  x*: integer

Output Schema:
  output: integer
>> c.add(x=10)
10
>> c.add(x=2)
12
```


## Notes on huggingface access

Sometimes, you might encounter errors accessing huggingface models, such as the following message when accessing `llama2`:
```text
Failed to create photon: 401 Client Error. (Request ID: Root=xxxxxxx)

Repo model meta-llama/Llama-2-7b-hf is gated. You must be authenticated to access it.
```
This means that you did not have access to the repo, or you did not set up huggingface access tokens. We'll detail how to do so below.


### Set up credentials to access huggingface

If you are running photons locally, you can do one of the following:
- set the token as an environmental variable, with `export HUGGING_FACE_HUB_TOKEN=your_token_here`.
- or, in your python environment, run the following command and login. Huggingface will store the credential in the local cache, usually `.huggingface/token`, for repeated usage:
```python
import huggingface_hub
huggingface_hub.login()
```

If you are running on the Trillium cloud remotely, the easiest approach is to use the `secret` feature of Trillium. You can safely store the huggingface token as a secret via CLI:
```shell
lep secret create -n HUGGING_FACE_HUB_TOKEN -v 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
```
(Don't worry, the above token is only an example and isn't active.)

You can verify the secret exists with `lep secret list`:
```shell
>> lep secret list
               Secrets               
┏━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━┓
┃ ID                     ┃ Value    ┃
┡━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━┩
│ HUGGING_FACE_HUB_TOKEN │ (hidden) │
└────────────────────────┴──────────┘
```

And when you launch a photon, add `--secret`:
```shell
lep photon run -n myphoton --secret HUGGING_FACE_HUB_TOKEN
```

