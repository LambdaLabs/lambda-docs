---
description: Learn how to deploy Llama 3.2 3B in a Kubernetes cluster.
tags:
  - api
  - kubernetes
  - llama
  - llm
---

# Deploying Llama 3.2 3B in a Kubernetes (K8s) cluster

## Introduction

In this tutorial, you'll:

1. Stand up a single-node Kubernetes cluster on an
   [on-demand instance :octicons-link-external-16:](https://lambdalabs.com/service/gpu-cloud){target="_blank"}
   using [K3s :octicons-link-external-16:](https://k3s.io/).
1. Install the
   [NVIDIA GPU Operator :octicons-link-external-16:](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/index.html){target="_blank"}
   so your cluster can use your instance's GPUs.
1. Deploy
   [Ollama :octicons-link-external-16:](https://ollama.com/){target="_blank"} in
   your cluster to serve the
   [Llama 3.2 3B model :octicons-link-external-16:](https://ai.meta.com/blog/llama-3-2-connect-2024-vision-edge-mobile-devices/){target="_blank"}.
1. Install the Ollama client.
1. Interact with the Llama 3.2 3B model.

!!! note

    You don't need a Kubernetes cluster to run Ollama and serve the Llama 3.2 3B
    model. Part of this tutorial is to demonstrate that it's possible to stand
    up a Kubernetes cluster on on-demand instances.

## Stand up a single-node Kubernetes cluster

1. If you haven't already, use the
   [dashboard :octicons-link-external-16:](https://cloud.lambdalabs.com/instances)
   or [Cloud API](#) to launch an instance. Then, SSH into your instance.

1. Install K3s (Kubernetes) by running:

    ```bash
    curl -sfL https://get.k3s.io | K3S_KUBECONFIG_MODE=644 sh -s - --default-runtime=nvidia
    ```

1. Verify that your Kubernetes cluster is ready by running:

    ```bash
    k3s kubectl get nodes
    ```

    You should see output similar to:

    ```{.text .no-copy}
    NAME              STATUS   ROLES                  AGE    VERSION
    104-171-203-164   Ready    control-plane,master   100s   v1.30.5+k3s1
    ```

1. Install socat by running:

    ```bash
    sudo apt -y install socat
    ```

    socat is needed to enable port forwarding in a later step.

## Install the NVIDIA GPU Operator

1. Install the NVIDIA GPU Operator in your Kubernetes cluster by running:

    ```bash
    cat <<EOF | k3s kubectl apply -f -
    apiVersion: v1
    kind: Namespace
    metadata:
      name: gpu-operator
    ---
    apiVersion: helm.cattle.io/v1
    kind: HelmChart
    metadata:
      name: gpu-operator
      namespace: gpu-operator
    spec:
      repo: https://helm.ngc.nvidia.com/nvidia
      chart: gpu-operator
      targetNamespace: gpu-operator
    EOF
    ```

1. In a few minutes, verify that your instance's GPUs are detected by your
   cluster by running:

    ```bash
    k3s kubectl describe nodes | grep nvidia.com
    ```

    You should see output similar to:

    ```{.text .no-copy}
    nvidia.com/cuda.driver-version.full=535.129.03
    nvidia.com/cuda.driver-version.major=535
    nvidia.com/cuda.driver-version.minor=129
    nvidia.com/cuda.driver-version.revision=03
    nvidia.com/cuda.driver.major=535
    nvidia.com/cuda.driver.minor=129
    nvidia.com/cuda.driver.rev=03
    nvidia.com/cuda.runtime-version.full=12.2
    nvidia.com/cuda.runtime-version.major=12
    nvidia.com/cuda.runtime-version.minor=2
    nvidia.com/cuda.runtime.major=12
    nvidia.com/cuda.runtime.minor=2
    nvidia.com/gfd.timestamp=1727461913
    nvidia.com/gpu-driver-upgrade-state=upgrade-done
    nvidia.com/gpu.compute.major=7
    nvidia.com/gpu.compute.minor=0
    nvidia.com/gpu.count=8
    nvidia.com/gpu.deploy.container-toolkit=true
    nvidia.com/gpu.deploy.dcgm=true
    nvidia.com/gpu.deploy.dcgm-exporter=true
    nvidia.com/gpu.deploy.device-plugin=true
    nvidia.com/gpu.deploy.driver=pre-installed
    nvidia.com/gpu.deploy.gpu-feature-discovery=true
    nvidia.com/gpu.deploy.node-status-exporter=true
    nvidia.com/gpu.deploy.operator-validator=true
    nvidia.com/gpu.family=volta
    nvidia.com/gpu.machine=Standard-PC-Q35-ICH9-2009
    nvidia.com/gpu.memory=16384
    nvidia.com/gpu.mode=compute
    nvidia.com/gpu.present=true
    nvidia.com/gpu.product=Tesla-V100-SXM2-16GB
    nvidia.com/gpu.replicas=1
    nvidia.com/gpu.sharing-strategy=none
    nvidia.com/mig.capable=false
    nvidia.com/mig.strategy=single
    nvidia.com/mps.capable=false
    nvidia.com/vgpu.present=false
    nvidia.com/gpu-driver-upgrade-enabled: true
    ```

    `nvidia.com/gpu.count=8` indicates that your cluster detects 8 GPUs.

    `nvidia.com/gpu.product=Tesla-V100-SXM2-16GB` indicates that the detected
    GPUs are Tesla V100 SXM2 16GB GPUs.

    !!! note

        In this tutorial, Ollama will only use 1 GPU.

## Deploy Ollama in your Kubernetes cluster

1. Start an Ollama server in your Kubernetes cluster by running:

    ```bash
    cat <<EOF | k3s kubectl apply -f -
    apiVersion: v1
    kind: Namespace
    metadata:
      name: ollama
    ---
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: ollama
      namespace: ollama
    spec:
      strategy:
        type: Recreate
      selector:
        matchLabels:
          name: ollama
      template:
        metadata:
          labels:
            name: ollama
        spec:
          containers:
          - name: ollama
            image: ollama/ollama:latest
            env:
            - name: PATH
              value: /usr/local/nvidia/bin:/usr/local/cuda/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
            - name: LD_LIBRARY_PATH
              value: /usr/local/nvidia/lib:/usr/local/nvidia/lib64
            - name: NVIDIA_DRIVER_CAPABILITIES
              value: compute,utility
            ports:
            - name: http
              containerPort: 11434
              protocol: TCP
            resources:
              limits:
                nvidia.com/gpu: 1
          tolerations:
          - key: nvidia.com/gpu
            operator: Exists
            effect: NoSchedule
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: ollama
      namespace: ollama
    spec:
      type: ClusterIP
      selector:
        name: ollama
      ports:
      - port: 80
        name: http
        targetPort: http
        protocol: TCP
    EOF
    ```

1. In a few minutes, run the following command to verify that the Ollama server
   is accepting connections and is using a GPU:

    ```
    kubectl logs -n ollama -l name=ollama
    ```

    You should see output similar to:

    ```{.text .no-copy}
    2024/09/27 18:51:55 routes.go:1153: INFO server config env="map[CUDA_VISIBLE_DEVICES: GPU_DEVICE_ORDINAL: HIP_VISIBLE_DEVICES: HSA_OVERRIDE_GFX_VERSION: HTTPS_PROXY: HTTP_PROXY: NO_PROXY: OLLAMA_DEBUG:false OLLAMA_FLASH_ATTENTION:false OLLAMA_GPU_OVERHEAD:0 OLLAMA_HOST:http://0.0.0.0:11434 OLLAMA_INTEL_GPU:false OLLAMA_KEEP_ALIVE:5m0s OLLAMA_LLM_LIBRARY: OLLAMA_LOAD_TIMEOUT:5m0s OLLAMA_MAX_LOADED_MODELS:0 OLLAMA_MAX_QUEUE:512 OLLAMA_MODELS:/root/.ollama/models OLLAMA_NOHISTORY:false OLLAMA_NOPRUNE:false OLLAMA_NUM_PARALLEL:0 OLLAMA_ORIGINS:[http://localhost https://localhost http://localhost:* https://localhost:* http://127.0.0.1 https://127.0.0.1 http://127.0.0.1:* https://127.0.0.1:* http://0.0.0.0 https://0.0.0.0 http://0.0.0.0:* https://0.0.0.0:* app://* file://* tauri://*] OLLAMA_SCHED_SPREAD:false OLLAMA_TMPDIR: ROCR_VISIBLE_DEVICES: http_proxy: https_proxy: no_proxy:]"
    time=2024-09-27T18:51:55.719Z level=INFO source=images.go:753 msg="total blobs: 0"
    time=2024-09-27T18:51:55.719Z level=INFO source=images.go:760 msg="total unused blobs removed: 0"
    time=2024-09-27T18:51:55.719Z level=INFO source=routes.go:1200 msg="Listening on [::]:11434 (version 0.3.12)"
    time=2024-09-27T18:51:55.720Z level=INFO source=common.go:49 msg="Dynamic LLM libraries" runners="[cpu_avx cpu_avx2 cuda_v11 cuda_v12 cpu]"
    time=2024-09-27T18:51:55.720Z level=INFO source=gpu.go:199 msg="looking for compatible GPUs"
    time=2024-09-27T18:51:55.942Z level=INFO source=types.go:107 msg="inference compute" id=GPU-d8c505a1-8af4-7ce4-517d-4f57fa576097 library=cuda variant=v12 compute=7.0 driver=12.2 name="Tesla V100-SXM2-16GB" total="15.8 GiB" available="15.5 GiB"
    ```

    The last line in the example output above shows that Ollama is using a
    single `Tesla V100-SXM2-16GB` GPU.

1. Start a tmux session by running:

    ```bash
    tmux
    ```

    Then, run the following command to make Ollama accessible from outside of
    your Kubernetes cluster:

    ```bash
    k3s kubectl -n ollama port-forward service/ollama 11434:80
    ```

    You should see output similar to:

    ```{.text .no-copy}
    Forwarding from 127.0.0.1:11434 -> 11434
    Forwarding from [::1]:11434 -> 11434
    ```

## Install the Ollama client

- Press ++ctrl++ + ++b++, then press ++c++ to open a new tmux window.

    Download and install the Ollama client by running:

    ```bash
    curl -L https://ollama.com/download/ollama-linux-amd64.tgz -o ollama-linux-amd64.tgz
    sudo tar -C /usr -xzf ollama-linux-amd64.tgz
    ```

## Serve and interact with the Llama 3.2 3B model

1. Serve the Llama 3.2 3B model using Ollama by running:

    ```bash
    ollama run llama3.2:3b-instruct-fp16
    ```

    You can interact with the model once you see the following prompt:

    ```{.text .no-copy}
    >>> Send a message (/? for help)
    ```

1. Test the model by entering a prompt, for example:

    ```
    What is machine learning? Explain like I'm five.
    ```

    You should see output similar to:

    ```{.text .no-copy}
    MACHINE LEARNING IS SO COOL!

    Imagine you have a toy box filled with different toys, like blocks, dolls, and cars. Now, imagine you want to teach a robot to pick up the toys from the box and put them away in their own boxes.

    At first, the robot doesn't know which toy goes where. So, you show it a few toys and say, "Hey, this is a block! Put it in the blocks' box!" The robot looks at the toy and says, "Okay, I think it's a block!"

    Then, you show it another toy and say, "This one is a doll! Put it in the dolls' box!" And so on.

    The robot keeps learning and trying to figure out which toys are which. It's like playing a game of "match me" with all the different toys!

    As the robot plays this game over and over, it gets better and better at recognizing the toys. Eventually, it can look at a new toy and say, "Oh, I know that one! That's a block! Put it in the blocks' box!"

    That's basically what machine learning is: teaching a computer to recognize patterns and make decisions on its own, just like the robot did with the toys!

    But instead of toys, computers use special math equations to learn from data (like pictures, words, or sounds). And instead of a toy box, they have big databases filled with lots of information.

    So, machine learning is all about helping computers get smarter and better at doing things on their own!
    ```
