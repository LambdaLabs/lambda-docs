---
description: >-
  Deploy a Llama 3 8B or 70B inference model using NVIDIA A100 or H100 Tensor
  Core GPUs in Lambda Cloud.
---

# Deploying a Llama 3 inference endpoint

Meta's Llama 3 large language models (LLMs) feature generative text models recognized for their state-of-the-art performance in common industry benchmarks.

This guide covers the deployment of a Meta [Llama 3](https://llama.meta.com/llama3/) inference endpoint using Lambda [On-Demand Cloud](https://lambdalabs.com/service/gpu-cloud). This tutorial uses the Llama 3 models hosted by [Hugging Face](https://huggingface.co/meta-llama/Meta-Llama-3-8B).

The model is available in 8B and 70B sizes:

| Model Size | Characteristics |
| :---- | :---- |
| 8B (8 billion parameters) | More efficient and accessible, suitable for tasks where resources are constrained. The 8B model requires a 1x A100 or H100 GPU node. |
| 70B (70 billion parameters) | Superior performance and capabilities ideal for complex or high-stakes applications. The 70B model requires an 8x A100 or H100 GPU node. |

### Prerequisites

This tutorial assumes the following prerequisites:

1. Lambda On-Demand Cloud instances appropriate for the Llama 3 model size you want to run.
    * Model 8B ([meta-llama/Meta-Llama-3-8B)](https://huggingface.co/meta-llama/Meta-Llama-3-8B) requires 1x A100 or H100 GPU node.
    * Model 70B ([meta-llama/Meta-Llama-3-70B)](https://huggingface.co/meta-llama/Meta-Llama-3-70B) requires 8x A100 or H100 GPU nodes.
1. A Hugging Face [user account](https://huggingface.co/join).
1. An approved [Hugging Face user access token](https://huggingface.co/docs/hub/en/security-tokens) that includes repository read permissions for the meta-llama-3 model repository you wish to use.

JSON outputs in this tutorial are formatted using [jq](https://jqlang.github.io/jq/).

### Set up the inference point

Once you have the appropriate Lambda On-Demand Cloud instances and Hugging Face permissions, begin by setting up an inference point.&#x20;

1. [Launch your Lambda On-Demand Cloud instance](https://cloud.lambdalabs.com/sign-up).
1. [Add or generate an SSH key](https://docs.lambdalabs.com/on-demand-cloud/dashboard#launch-instances) to access the instance.
1. SSH into your instance.
1. Create a dedicated python environment.

    === "Llama3 8b"

         ```
         python3 -m venv Meta-Llama-3-8B
         source Meta-Llama-3-8B/bin/activate
         python3 -m pip install vllm==0.4.3 huggingface-hub==0.23.2 torch==2.3.0 numpy==1.26.4
         ```

    === "Llama3 70b"

         ```
         python3 -m venv Meta-Llama-3-70B
         source Meta-Llama-3-70B/bin/activate
         python3 -m pip install vllm==0.4.3 huggingface-hub==0.23.2 torch==2.3.0 numpy==1.26.4
         ```

1. Log in to Hugging Face:

    ```
    huggingface-cli login
    ```

1. Start the model server (download/cache as necessary).&#x20;

    === "Llama3 8B"

        ```
        python3 -m vllm.entrypoints.openai.api_server \
          --host=0.0.0.0 \
          --port=8000 \
          --model=meta-llama/Meta-Llama-3-8B &> api_server.log & 
        ```

    === "Llama3 70B"

        ```
         python3 -m vllm.entrypoints.openai.api_server \
          --host=0.0.0.0 \
          --port=8000 \
          --model=meta-llama/Meta-Llama-3-70B \
          --tensor-parallel-size 8 &> api_server.log &

        // The API server may take several minutes to start.
        ```

### Interact with the Model

The following request delivers language prompts to the Llama 3 model:

=== "Llama3 8B"

    ```
    curl -X POST http://localhost:8000/v1/completions \
         -H "Content-Type: application/json" \
         -d '{
               "prompt": "What is the name of the capital of France?",
               "model": "meta-llama/Meta-Llama-3-8B",
               "temperature": 0.0,
               "max_tokens": 1   // sets the response length
             }'
    ```

=== "Llama3 70B"

    ```
    curl -X POST http://localhost:8000/v1/completions \
         -H "Content-Type: application/json" \
         -d '{
               "prompt": "What is the name of the capital of France?",
               "model": "meta-llama/Meta-Llama-3-70B",
               "temperature": 0.0,
               "max_tokens": 1   // sets the response length
             }'
    ```

Llama 3 responds to requests in the following format:

=== "Llama3 8B"

    ```
    {
      "id": "cmpl-d898e2089b7b4855b48e00684b921c95",
      "object": "text_completion",
      "created": 1718221710,
      "model": "meta-llama/Meta-Llama-3-8B",
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

=== "Llama3 70B"

    ```
    {
      "id": "cmpl-d898e2089b7b4855b48e00684b921c95",
      "object": "text_completion",
      "created": 1718221710,
      "model": "meta-llama/Meta-Llama-3-70B",
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
