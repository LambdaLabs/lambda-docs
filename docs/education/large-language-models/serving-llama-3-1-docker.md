---
description: How to serve the Llama 3.1 8B and 70B models using Lambda Cloud on-demand instances
tags:
  - docker
  - llama
  - llm
---

# Serving the Llama 3.1 8B and 70B models using Lambda Cloud on-demand instances

This tutorial shows you how to use a [Lambda
Cloud](https://lambdalabs.com/service/gpu-cloud) 1x or 8x A100 or H100
on-demand instance to serve the Llama 3.1 8B and 70B models. You'll serve the
model using [vLLM running inside of a Docker
container](https://docs.vllm.ai/en/latest/serving/deploying_with_docker.html).

## Start the vLLM API server

If you haven't already, use the
[dashboard](https://cloud.lambdalabs.com/instances) or [Cloud
API](https://docs.lambdalabs.com/on-demand-cloud/cloud-api) to launch an
instance. Then, SSH into your instance.

Run:

```bash
export HF_TOKEN=HF-TOKEN HF_HOME="/home/ubuntu/.cache/huggingface" MODEL_REPO=meta-llama/MODEL
```

Replace **HF-TOKEN** with your Hugging Face User Access Token.

Replace **MODEL** with:

- If you're serving the 8B model:
  ```
  Meta-Llama-3.1-8B-Instruct
  ```
- If you're serving the 70B model:
  ```
  Meta-Llama-3.1-70B-Instruct
  ```

These commands set environment variables needed for this tutorial.

!!! note

    The 70B model requires an 8x A100 or H100.

Start a tmux session by running `tmux`.

If you're serving the 8B model, run:

```bash
sudo docker run \
  --gpus all \
  --ipc=host \
  -v "${HF_HOME}":/root/.cache/huggingface \
  -p 8000:8000 \
  --env "HUGGING_FACE_HUB_TOKEN=${HF_TOKEN}" \
  vllm/vllm-openai --model "${MODEL_REPO}" \
    --disable-log-requests
```

If you're serving the 70B model using an 8x A100 or H100 instance (required),
instead run:

```bash
sudo docker run \
  --gpus all \
  --ipc=host \
  -v "${HF_HOME}":/root/.cache/huggingface \
  -p 8000:8000 \
  --env "HUGGING_FACE_HUB_TOKEN=${HF_TOKEN}" \
  vllm/vllm-openai --model "${MODEL_REPO}" \
    --disable-log-requests \
    --tensor-parallel-size 8
```

Both commands, above:

- Download the model you're serving.
- Start vLLM's API server with the chosen model.

!!! note

    The difference betweeen the two commands, above, is that the second
    command enables the tensor parallel strategy to use 8x GPUs. [See vLLM's docs to
    learn more about the distributed inference
    strategies](https://docs.vllm.ai/en/latest/serving/distributed_serving.html).

The vLLM API server is running once you see:

```
INFO 08-01 19:11:07 api_server.py:292] Available routes are:
INFO 08-01 19:11:07 api_server.py:297] Route: /openapi.json, Methods: GET, HEAD
INFO 08-01 19:11:07 api_server.py:297] Route: /docs, Methods: GET, HEAD
INFO 08-01 19:11:07 api_server.py:297] Route: /docs/oauth2-redirect, Methods: GET, HEAD
INFO 08-01 19:11:07 api_server.py:297] Route: /redoc, Methods: GET, HEAD
INFO 08-01 19:11:07 api_server.py:297] Route: /health, Methods: GET
INFO 08-01 19:11:07 api_server.py:297] Route: /tokenize, Methods: POST
INFO 08-01 19:11:07 api_server.py:297] Route: /detokenize, Methods: POST
INFO 08-01 19:11:07 api_server.py:297] Route: /v1/models, Methods: GET
INFO 08-01 19:11:07 api_server.py:297] Route: /version, Methods: GET
INFO 08-01 19:11:07 api_server.py:297] Route: /v1/chat/completions, Methods: POST
INFO 08-01 19:11:07 api_server.py:297] Route: /v1/completions, Methods: POST
INFO 08-01 19:11:07 api_server.py:297] Route: /v1/embeddings, Methods: POST
INFO:     Started server process [1]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)
```

## Test the vLLM API server

To test that the API server is serving the Llama 3.1 model:

Press ++ctrl++ + ++b++, then press ++c++ to open a new tmux window.

Then, run:

```bash
curl -X POST http://localhost:8000/v1/completions \
     -H "Content-Type: application/json" \
     -d "{
           \"prompt\": \"What is the name of the capital of France?\",
           \"model\": \"${MODEL_REPO}\",
           \"temperature\": 0.0,
           \"max_tokens\": 1
         }"
```

You should see output similar to:

```json
{"id":"cmpl-d3a33498b5d74d9ea09a7c256733b8df","object":"text_completion","created":1722545598,"model":"meta-llama/Meta-Llama-3.1-70B-Instruct","choices":[{"index":0,"text":" Paris","logprobs":null,"finish_reason":"length","stop_reason":null}],"usage":{"prompt_tokens":11,"total_tokens":12,"completion_tokens":1}}
```

!!! tip

    You can make the output more human-readable using jq. To do this, first
    install jq by running:

    ```bash
    sudo apt update && sudo apt install -y jq
    ```

    Then, append `| jq .` to the `curl` command, above.

    The complete command should be:

    ```bash
    curl -X POST http://localhost:8000/v1/completions \
         -H "Content-Type: application/json" \
         -d "{
               \"prompt\": \"What is the name of the capital of France?\",
               \"model\": \"${MODEL_REPO}\",
               \"temperature\": 0.0,
               \"max_tokens\": 1
             }" | jq .
    ```

    The output should now look similar to:

    ```json
    {
      "id": "cmpl-529d01c83069409fa5c166e1d137e21e",
      "object": "text_completion",
      "created": 1722545913,
      "model": "meta-llama/Meta-Llama-3.1-70B-Instruct",
      "choices": [
        {
          "index": 0,
          "text": " Paris",
          "logprobs": null,
          "finish_reason": "length",
          "stop_reason": null
        }
      ],
      "usage": {
        "prompt_tokens": 11,
        "total_tokens": 12,
        "completion_tokens": 1
      }
    }
    ```
