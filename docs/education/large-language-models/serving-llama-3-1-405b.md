---
description: >-
  Deploy a Llama 3.1 405B inference model using NVIDIA H100 Tensor Core GPUs in
  Lambda 1-Click Clusters
---

# Serving Llama 3.1 405B on a Lambda 1-Click Cluster

In this tutorial, you'll learn how to use a 1-Click Cluster (1CC) to serve the [Meta Llama 3.1 405B model](https://huggingface.co/meta-llama/Meta-Llama-3.1-405B) using [vLLM](https://docs.vllm.ai/en/latest/index.html) with pipeline parallelism.

## Prerequisites

For this tutorial, you need:&#x20;

* A [Lambda Cloud account](https://cloud.lambdalabs.com/sign-up).
* A [1-Click Cluster](https://lambdalabs.com/service/gpu-cloud/1-click-clusters).&#x20;
* A [Hugging Face](https://huggingface.co/) account to download the Llama 3.1 405B model.&#x20;
* A [User Access Token](https://huggingface.co/docs/hub/en/security-tokens) with the **Read** role.
* Before you can download the Llama 3.1 405B model, you need to review and accept the model's license agreement. Once you accept the agreement, a request to access the repository will be submitted for approval; approval tends to be fast. You can see the status of the request in your [Hugging Face account settings](https://huggingface.co/settings/gated-repos).

### Download the Llama 3.1 405B model and set up a head node

First, follow the [instructions for accessing your 1CC](https://docs.lambdalabs.com/1-click-clusters/getting-started#accessing-your-1-click-cluster), which includes steps to set up SSH.

Then SSH into one of your 1CC GPU nodes. You can find the node names in your [1-Click Clusters dashboard](https://cloud.lambdalabs.com/one-click-clusters/running). You’ll use this GPU node as a head node for cluster management.&#x20;

On the head node, set environment variables needed for this tutorial by running:

```bash
export HEAD_IP=HEAD-IP
export SHARED_DIR=/home/ubuntu/FILE-SYSTEM-NAME
export HF_TOKEN=HF-TOKEN
export HF_HOME="${SHARED_DIR}/.cache/huggingface"
export MODEL_REPO=meta-llama/Meta-Llama-3.1-405B-Instruct
```

Replace `HEAD-IP` with the IP address of the head node. You can obtain the IP address from the [1-Click Clusters dashboard](https://cloud.lambdalabs.com/one-click-clusters/running).

Replace `FILE-SYSTEM-NAME` with the name of your 1CC's persistent storage file system.

Replace `HF-TOKEN` with your Hugging Face User Access Token.

Then, run:

```bash
mkdir -p "${HF_HOME}"
python3 -m venv llama-3.1
source llama-3.1/bin/activate
pip install -U huggingface_hub[cli]
huggingface-cli login --token "${HF_TOKEN}"
huggingface-cli download "${MODEL_REPO}"
```

These commands:

1. Create and activate a [Python virtual environment](https://docs.lambdalabs.com/software/virtual-environments-and-docker-containers#creating-a-python-virtual-environment) on the head node for this tutorial.
2. Download the Llama 3.1 405B model to your 1CC's persistent storage [file system](https://lambdalabs.atlassian.net/wiki/spaces/LT/pages/edit-v2/709328915).

!!! note

    The Llama 3.1 405B model is about 2.3TB in size and can take several hours to download.

Still on the head node, start a tmux session by running `tmux`.

Then, run:

```bash
curl -o "${SHARED_DIR}/run_cluster.sh" https://raw.githubusercontent.com/vllm-project/vllm/main/examples/run_cluster.sh
sudo bash "${SHARED_DIR}/run_cluster.sh" \
    vllm/vllm-openai \
    "${HEAD_IP}" \
    --head "${HF_HOME}" \
    --privileged -e NCCL_IB_HCA=^mlx5_0
```

These commands:

1. Download a helper script to your shared persistent storage file system to [set up vLLM for multi-node inference and serving](https://docs.vllm.ai/en/latest/serving/distributed\_serving.html#multi-node-inference-and-serving).
2. Run the script to start a [Ray cluster](https://docs.ray.io/en/latest/cluster/getting-started.html) for serving the Llama 3.1 405B model using vLLM. The Ray cluster uses your 1CC's InfiniBand fabric for optimal performance.

### Connect another GPU node to the head node

Next, you'll connect another of your 1CC's GPUs nodes to the head node. This other GPU node will be referred to below as the worker node.&#x20;

In a new terminal, SSH into the worker node, then set environment variables needed for this tutorial by running:

```bash
export HEAD_IP=HEAD-IP
export SHARED_DIR=/home/ubuntu/FILE-SYSTEM-NAME
export HF_HOME="${SHARED_DIR}/.cache/huggingface"
```

Replace `HEAD-IP` with the IP address of the head node.

Replace `FILE-SYSTEM-NAME` with the name of your 1CC's persistent storage file system.

Run `tmux` to start a new tmux session. Then, run:

```bash
sudo bash "${SHARED_DIR}/run_cluster.sh" \
    vllm/vllm-openai \
    "${HEAD_IP}" \
    --worker \
    "${HF_HOME}" \
    --privileged -e NCCL_IB_HCA=^mlx5_0
```

This command connects the worker node to the head node.

### Check the status of the Ray cluster

On the worker node, open a new tmux window (press **Ctrl + B**, then press **C**). Then, run:

```bash
sudo docker exec -it node /bin/bash
```

Check the status of the Ray cluster by running:

```bash
ray status
```

This command produces output similar to the following:

```
======== Autoscaler status: 2024-07-25 00:20:50.831620 ========
Node status
---------------------------------------------------------------
Active:
 1 node_d86d9f0f1894c2e463d8168530f6745e32beb08ddf3b908d229d8527
 1 node_37af7f860c4bab2e035b5a55ba06e2e49dba9fa891d65f8264648804
Pending:
 (no pending nodes)
Recent failures:
 (no failures)

Resources
---------------------------------------------------------------
Usage:
 0.0/416.0 CPU
 16.0/16.0 GPU (16.0 used of 16.0 reserved in placement groups)
 0B/3.43TiB memory
 0B/19.46GiB object_store_memory

Demands:
 (no resource demands)
```

This output shows two active nodes (the head node and the worker node) and 16 GPUs in the Ray cluster.

### Serve the Llama 3.1 405B model

After verifying the server status, you can start to serve the model.&#x20;

```bash
ls /root/.cache/huggingface/hub/models--meta-llama--Meta-Llama-3.1-405B-Instruct/snapshots
```

This command returns a snapshot with a name similar to:&#x20;

```
e04e3022cdc89bfed0db69f5ac1d249e21ee2d30
```

Now, run the following command to begin serving the Llama 3.1 405B model:

```bash
vllm serve "/root/.cache/huggingface/hub/models--meta-llama--Meta-Llama-3.1-405B-Instruct/snapshots/SNAPSHOT" --tensor-parallel-size 8 --pipeline-parallel-size 2
```

Replace `SNAPSHOT` with the name of the Llama 3.1 405B model snapshot you retrieved earlier.&#x20;

!!! note

    It can take 15 minutes or more before the model is loaded onto the GPUs and ready to be served.

You should begin seeing output similar to:

```
INFO 07-25 04:17:41 api_server.py:219] vLLM API server version 0.5.3.post1
INFO 07-25 04:17:41 api_server.py:220] args: Namespace(model_tag='/root/.cache/huggingface/hub/models--meta-llama--Meta-Llama-3.1-405B-Instruct/snapshots/e04e3022cdc89bfed0db69f5ac1d249e21ee2d30', host=None, port=8000, uvicorn_log_level='info', allow_credentials=False, allowed_origins=['*'], allowed_methods=['*'], allowed_headers=['*'], api_key=None, lora_modules=None, prompt_adapters=None, chat_template=None, response_role='assistant', ssl_keyfile=None, ssl_certfile=None, ssl_ca_certs=None, ssl_cert_reqs=0, root_path=None, middleware=[], model='/root/.cache/huggingface/hub/models--meta-llama--Meta-Llama-3.1-405B-Instruct/snapshots/e04e3022cdc89bfed0db69f5ac1d249e21ee2d30', tokenizer=None, skip_tokenizer_init=False, revision=None, code_revision=None, tokenizer_revision=None, tokenizer_mode='auto', trust_remote_code=False, download_dir=None, load_format='auto', dtype='auto', kv_cache_dtype='auto', quantization_param_path=None, max_model_len=None, guided_decoding_backend='outlines', distributed_executor_backend=None, worker_use_ray=False, pipeline_parallel_size=2, tensor_parallel_size=8, max_parallel_loading_workers=None, ray_workers_use_nsight=False, block_size=16, enable_prefix_caching=False, disable_sliding_window=False, use_v2_block_manager=False, num_lookahead_slots=0, seed=0, swap_space=4, cpu_offload_gb=0, gpu_memory_utilization=0.9, num_gpu_blocks_override=None, max_num_batched_tokens=None, max_num_seqs=256, max_logprobs=20, disable_log_stats=False, quantization=None, rope_scaling=None, rope_theta=None, enforce_eager=False, max_context_len_to_capture=None, max_seq_len_to_capture=8192, disable_custom_all_reduce=False, tokenizer_pool_size=0, tokenizer_pool_type='ray', tokenizer_pool_extra_config=None, enable_lora=False, max_loras=1, max_lora_rank=16, lora_extra_vocab_size=256, lora_dtype='auto', long_lora_scaling_factors=None, max_cpu_loras=None, fully_sharded_loras=False, enable_prompt_adapter=False, max_prompt_adapters=1, max_prompt_adapter_token=0, device='auto', scheduler_delay_factor=0.0, enable_chunked_prefill=None, speculative_model=None, num_speculative_tokens=None, speculative_draft_tensor_parallel_size=None, speculative_max_model_len=None, speculative_disable_by_batch_size=None, ngram_prompt_lookup_max=None, ngram_prompt_lookup_min=None, spec_decoding_acceptance_method='rejection_sampler', typical_acceptance_sampler_posterior_threshold=None, typical_acceptance_sampler_posterior_alpha=None, disable_logprobs_during_spec_decoding=None, model_loader_extra_config=None, ignore_patterns=[], preemption_mode=None, served_model_name=None, qlora_adapter_name_or_path=None, otlp_traces_endpoint=None, engine_use_ray=False, disable_log_requests=False, max_log_len=None, dispatch_function=<function serve at 0x7de812d13520>)
INFO 07-25 04:17:41 config.py:715] Defaulting to use ray for distributed inference
```

The Llama 3.1 405B model is ready to be served once you see output similar to:

```
INFO:     Started server process [24469]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)
```

### Test the Llama 3.1 405B model

Still on the worker node, open a new tmux window (**Ctrl + B**, then press **C**).

Then, run:

```bash
python3 -m venv llama-3.1
source llama-3.1/bin/activate
pip install -U openai
curl -o ${SHARED_DIR}/inference_test.py 'https://raw.githubusercontent.com/vllm-project/vllm/main/examples/openai_chat_completion_client.py'
```

These commands:

1. Create and activate a [Python virtual environment](https://docs.lambdalabs.com/software/virtual-environments-and-docker-containers#creating-a-python-virtual-environment) on the worker node.
2. Download vLLM’s example Open AI chat completion client.

Finally, to test the Llama 3.1 405B model, run:

```bash
python3 ${SHARED_DIR}/inference_test.py
```

This command produces output similar to:

```
Chat completion results:
ChatCompletion(id='chat-8eba7fa7e2f7442aafa82a1683bfc77f', choices=[Choice(finish_reason='stop', index=0, logprobs=None, message=ChatCompletionMessage(content='The 2020 World Series was played at Globe Life Field in Arlington, Texas. This was a neutral site due to COVID-19 restrictions and was also referred to as a "bubble" environment.', role='assistant', function_call=None, tool_calls=[]), stop_reason=None)], created=1721884178, model='/root/.cache/huggingface/hub/models--meta-llama--Meta-Llama-3.1-405B-Instruct/snapshots/e04e3022cdc89bfed0db69f5ac1d249e21ee2d30', object='chat.completion', service_tier=None, system_fingerprint=None, usage=CompletionUsage(completion_tokens=41, prompt_tokens=59, total_tokens=100))
```

### Acknowledgement

We'd like to thank the [vLLM](https://github.com/vllm-project/vllm) team for their partnership in developing this guide and their pioneering work in streamlining LLM serving.
