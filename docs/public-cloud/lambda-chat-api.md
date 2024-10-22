---
description: Using the Lambda Chat Completions API
tags:
  - api
  - llama
  - llm
---

# Using the Lambda Chat Completions API

The Lambda Chat Completions API enables you to use the
[Llama 3.1 405B Instruct large language model (LLM)](https://llama.meta.com/){ .external target="_blank" },
and fine-tuned versions such as Nous Research's Hermes 3 and Liquid AI's LFM
40.3B MoE (Mixture of Experts), without the need to set up your own vLLM API
server on an
[on-demand instance](https://lambdalabs.com/service/gpu-cloud){ .external target="_blank" }
or
[1-Click Cluster (1CC)](https://lambdalabs.com/service/gpu-cloud/1-click-clusters){ .external target="_blank" }.

!!! note

    Try [Lambda Chat](https://lambda.chat/chatui/){ .external target="_blank" }!

Since the Lambda Chat Completions API is compatible with the
[OpenAI API](https://platform.openai.com/docs/overview){ .external target="_blank" },
you can use it as a drop-in replacement for applications currently using the
OpenAI API. See, for example,
[our guide on integrating Lambda Chat into VS Code](../education/programming/vs-code-lambda-chat.md).

The Lambda Chat Completions API implements endpoints for:

- [Creating chat completions](#creating-chat-completions) (`/chat/completions`)
- [Creating completions](#creating-completions) (`/completions`)
- [Listing models](#listing-models) (`/models`)

Currently, the following models are available:

- `deepseek-coder-v2-lite-instruct`
- `dracarys2-72b-instruct`
- `hermes3-405b`
- `hermes3-405b-fp8-128k`
- `hermes3-70b`
- `hermes3-8b`
- `lfm-40b`
- `llama3.1-405b-instruct-fp8`
- `llama3.1-70b-instruct-fp8`
- `llama3.1-8b-instruct`
- `llama3.2-3b-instruct`
- `llama3.1-nemotron-70b-instruct`

!!! note

    If a request using the `hermes3-405b` model is made with a
    context length greater than 18K, the request will fall back to using the
    `hermes-3-llama-3.1-405b-fp8-128k` model.

To use the Lambda Chat Completions API, first
[generate a Cloud API key from the dashboard](../public-cloud/on-demand/dashboard.md#generate-api-keys).
You can also use a Cloud API key that you've already generated.

In the examples below:

- Replace `<MODEL>` with one of the models listed above.
- Replace `<API-KEY>` with your actual Cloud API key.

## Creating chat completions

The `/chat/completions` endpoint takes a list of messages that make up a
conversation, then outputs a response.

=== "Curl"

    First, create a file named `messages.json` that contains
    [the necessary and any optional parameters](https://platform.openai.com/docs/api-reference/chat/create){ .external target="_blank" }.
    For example:

    ```json
    {
      "model": "<MODEL>",
      "messages": [
        {
          "role": "system",
          "content": "You are a helpful assistant named Hermes, made by Nous Research."
        },
        {
          "role": "user",
          "content": "Who won the world series in 2020?"
        },
        {
          "role": "assistant",
          "content": "The Los Angeles Dodgers won the World Series in 2020."
        },
        {
          "role": "user",
          "content": "Where was it played?"
        }
      ]
    }
    ```

    Then, run:

    ```bash
    curl https://api.lambdalabs.com/v1/chat/completions -d @messages.json -H "Authorization: Bearer <API-KEY>" -H "Content-Type: application/json" | jq .
    ```

    You should see output similar to:

    ```json
    {
      "id": "chat-f569652bd6a64e77b01dbb1955832998",
      "choices": [
        {
          "finish_reason": "stop",
          "index": 0,
          "message": {
            "content": "The 2020 World Series was played at Globe Life Field in Arlington, Texas. Due to the COVID-19 pandemic, the entire series was held at a neutral location to minimize travel and potential exposure to the virus.",
            "role": "assistant",
            "tool_calls": null,
            "function_call": null
          }
        }
      ],
      "created": 1723728526,
      "model": "hermes3-405b",
      "object": "chat.completion",
      "system_fingerprint": null,
      "usage": {
        "completion_tokens": 45,
        "prompt_tokens": 58,
        "total_tokens": 103
      },
      "service_tier": null
    }
    ```

=== "Python"

    First,
    [create and activate a Python virtual environment](../education/programming/virtual-environments-containers.md#creating-a-python-virtual-environment).
    Then, install the
    [OpenAI Python API library](https://pypi.org/project/openai/){ .external target="_blank" }.

    Run, for example:

    ```python
    from openai import OpenAI

    openai_api_key = "<API-KEY>"
    openai_api_base = "https://api.lambdalabs.com/v1"

    client = OpenAI(
        api_key=openai_api_key,
        base_url=openai_api_base,
    )

    model = "<MODEL>"

    chat_completion = client.chat.completions.create(
        messages=[{
            "role": "system",
            "content": "You are a helpful assistant named Hermes, made by Nous Research."
        }, {
            "role": "user",
            "content": "Who won the world series in 2020?"
        }, {
            "role":
            "assistant",
            "content":
            "The Los Angeles Dodgers won the World Series in 2020."
        }, {
            "role": "user",
            "content": "Where was it played?"
        }],
        model=model,
    )

    print(chat_completion)
    ```

    You should see output similar to:

    ```
    ChatCompletion(id='chat-e489d950acaa41deb02cb794d1ecfe6b', choices=[Choice(finish_reason='stop', index=0, logprobs=None, message=ChatCompletionMessage(content='The 2020 World Series was played at Globe Life Field in Arlington, Texas. Due to the COVID-19 pandemic, the entire series was held at a neutral site to limit travel and potential exposure to the virus.', refusal=None, role='assistant', function_call=None, tool_calls=None))], created=1723738002, model='hermes3-405b', object='chat.completion', service_tier=None, system_fingerprint=None, usage=CompletionUsage(completion_tokens=45, prompt_tokens=58, total_tokens=103))
    ```

## Creating completions

The `/completions` endpoint takes a single text string (a prompt) as input,
then outputs a response. In comparison, the `/chat/completions` endpoint takes
a list of messages as input.

To use the `/completions` endpoint:

=== "Curl"

    First, create a file named `prompt.json` that contains
    [the necessary and any optional parameters](https://platform.openai.com/docs/api-reference/completions){ .external target="_blank" }.
    For example:

    ```json
    {
      "model": "<MODEL>",
      "prompt": "Computers are",
      "temperature": 0
    }
    ```

    Then, run:

    ```bash
    curl https://api.lambdalabs.com/v1/completions -d @prompt.json -H "Authorization: Bearer <API-KEY>" -H "Content-Type: application/json" | jq .
    ```

    You should see output similar to:

    ```json
    {
      "id": "cmpl-0ed3e4f07bf242bab6d4ec67c48e0e83",
      "object": "text_completion",
      "created": 1723487710,
      "model": "hermes3-405b",
      "choices": [
        {
          "stop_reason": null,
          "finish_reason": "stop",
          "index": 0,
          "text": " a part of our daily lives, and we use them for various purposes, including work, entertainment, and communication. However, like any other electronic device, computers are prone to malfunctions and errors. When your computer starts acting up, it can be frustrating, especially if you don’t know how to fix the problem. In this article, we will discuss some common computer problems and how to troubleshoot them.\n1. Slow Performance\nOne of the most common computer problems is slow performance. If your computer is running slowly, there could be several reasons why. Here are some steps you can take to troubleshoot the issue:\nCheck your hard drive space: If your hard drive is almost full, it can slow down your computer. Free up some space by deleting unnecessary files or moving them to an external hard drive.\nClose unused programs: If you have too many programs running at the same time, it can slow down your computer. Close any programs that you’re not using.\nRun a virus scan: Malware and viruses can slow down your computer. Run a virus scan to check for any malicious software.\nUpgrade your hardware: If your computer is old, it may not have the necessary hardware to run newer programs efficiently. Consider upgrading your RAM or processor.\n2. Blue Screen of Death\nThe Blue Screen of Death (BSOD) is a common error that occurs when your computer encounters a critical system error. When this happens, your computer will restart, and you may lose any unsaved work. Here are some steps you can take to troubleshoot the issue:\nCheck for updates: Make sure your operating system and drivers are up to date.\nRun a memory test: A faulty RAM module can cause the BSOD. Run a memory test to check for any errors.\nCheck your hardware: Loose cables, overheating, and faulty hardware can all cause the BSOD. Check your computer’s hardware and make sure everything is properly connected and functioning.\n3. Internet Connectivity Issues\nIf you’re having trouble connecting to the internet, there could be several reasons why. Here are some steps you can take to troubleshoot the issue:\nCheck your router: Make sure your router is properly connected and turned on. If it’s not working, try restarting it.\nCheck your network settings: Make sure your computer is set to connect to the correct network and that your network settings are correct.\nRun a network troubleshooter: Most operating systems have a built-in network troubleshooter that can help diagnose and fix connectivity issues.\n4. Printer Problems\nPrinter problems are another common issue that computer users face. If your printer isn’t working, here are some steps you can take to troubleshoot the issue:\nCheck your printer queue: If there are too many print jobs in the queue, it can cause your printer to stop working. Clear the print queue and try again.\nCheck your printer drivers: Make sure your printer drivers are up to date. If they’re not, download and install the latest drivers from the manufacturer’s website.\nCheck your printer settings: Make sure your printer settings are correct. If they’re not, adjust them and try again.\n5. Audio Issues\nIf you’re having trouble with your computer’s audio, there could be several reasons why. Here are some steps you can take to troubleshoot the issue:\nCheck your audio settings: Make sure your audio settings are correct and that your speakers or headphones are properly connected.\nUpdate your audio drivers: If your audio drivers are outdated, it can cause audio issues. Download and install the latest drivers from the manufacturer’s website.\nCheck for muted applications: If you’re not hearing any sound from a specific application, make sure it’s not muted.\nIn conclusion, computer problems can be frustrating, but with the right troubleshooting steps, you can often fix the issue yourself. If you’re still having trouble, consider contacting a professional for help.\n*Note: This is a sample article and may not reflect the most up-to-date information or best practices. Always consult with a professional for the most accurate and relevant advice.*",
          "logprobs": null
        }
      ],
      "usage": {
        "completion_tokens": 816,
        "prompt_tokens": 4,
        "total_tokens": 820
      },
      "system_fingerprint": null
    }
    ```

=== "Python"

    First,
    [create and activate a Python virtual environment](../education/programming/virtual-environments-containers.md#creating-a-python-virtual-environment).
    Then, install the
    [OpenAI Python API library](https://pypi.org/project/openai/){ .external target="_blank" }.

    Run, for example:

    ```python
    from openai import OpenAI

    openai_api_key = "<API-KEY>"
    openai_api_base = "https://api.lambdalabs.com/v1"

    client = OpenAI(
        api_key=openai_api_key,
        base_url=openai_api_base,
    )

    model = "<MODEL>"

    response = client.completions.create(
      prompt="Computers are",
      temperature=0,
      model=model,
    )

    print(response)
    ```

    You should see output similar to:

    ```
    Completion(id='cmpl-bed15d67c6894588bc0292c1cc5ed28d', choices=[CompletionChoice(finish_reason='stop', index=0, logprobs=None, text=' a part of our everyday lives. They are used in homes, schools, businesses, and many other places. Computers have changed the way we live, work, and play. They have made many tasks easier and more efficient. However, computers can also be frustrating and challenging to use at times. In this article, we will explore the pros and cons of computers.\nPros of Computers:\n1. Increased Productivity: Computers have made many tasks easier and more efficient. They can perform complex calculations, store large amounts of data, and automate repetitive tasks. This has led to increased productivity in many industries.\n\n2. Improved Communication: Computers have revolutionized the way we communicate. With email, instant messaging, and video conferencing, we can communicate with people all over the world instantly.\n\n3. Access to Information: The internet has made a vast amount of information available to us at our fingertips. We can search for information on any topic and find answers to our questions quickly and easily.\n\n4. Entertainment: Computers have also changed the way we entertain ourselves. We can play games, watch movies, listen to music, and read books all on our computers.\n\n5. Education: Computers have had a significant impact on education. They have made it possible for students to learn from anywhere in the world and have access to a wealth of educational resources.\n\nCons of Computers:\n1. Dependence: As we become more reliant on computers, we may find it difficult to function without them. This dependence can be problematic if there is a power outage or if our computers break down.\n\n2. Health Issues: Prolonged use of computers can lead to health issues such as eye strain, headaches, and back pain. Sitting for long periods can also contribute to obesity and other health problems.\n\n3. Security Risks: Computers are vulnerable to security risks such as viruses, malware, and hacking. These risks can compromise our personal information and lead to identity theft.\n\n4. Social Isolation: While computers have improved communication, they can also lead to social isolation. People may spend more time interacting with their computers than with other people, leading to feelings of loneliness and disconnection.\n\n5. Cost: Computers can be expensive to purchase and maintain. Upgrading hardware and software can also be costly.\n\nIn conclusion, computers have both pros and cons. They have made many aspects of our lives easier and more efficient, but they also come with challenges and risks. It is essential to use computers responsibly and to be aware of the potential drawbacks. By doing so, we can maximize the benefits of computers while minimizing the negative impacts.1) + 1\n    return n\n\ndef main():\n    n = int(input())\n    print(fibonacci(n))\n\nif __name__ == "__main__":\n    main()\n```\n\n## Explanation\n\nThe `fibonacci` function takes an integer `n` as input and returns the `n`-th Fibonacci number. The function uses recursion to calculate the Fibonacci number.\n\nThe base cases for the recursion are:\n- If `n` is 0, the function returns 0.\n- If `n` is 1, the function returns 1.\n\nFor any other value of `n`, the function recursively calls itself with `n-1` and `n-2` and returns the sum of the results.\n\nIn the `main` function, we read an integer `n` from the user using `input()` and convert it to an integer using `int()`. We then call the `fibonacci` function with `n` and print the result.\n\nThe `if __name__ == "__main__":` condition ensures that the `main` function is only executed when the script is run directly, and not when it is imported as a module.\n\n## Example\n\nLet\'s say the user inputs the number 6. The program will output:\n```\n8\n```\n\nThis is because the 6th Fibonacci number is 8 (0, 1, 1, 2, 3, 5, 8).\n\nThe recursive calls will be as follows:\n- `fibonacci(6)` calls `fibonacci(5)` and `fibonacci(4)`\n  - `fibonacci(5)` calls `fibonacci(4)` and `fibonacci(3)`\n    - `fibonacci(4)` calls `fibonacci(3)` and `fibonacci(2)`\n      - `fibonacci(3)` calls `fibonacci(2)` and `fibonacci(1)`\n        - `fibonacci(2)` calls `fibonacci(1)` and `fibonacci(0)`\n          - `fibonacci(1)` returns 1\n          - `fibonacci(0)` returns 0\n        - `fibonacci(1)` returns 1\n      - `fibonacci(2)` returns 1\n    - `fibonacci(3)` returns 2\n  - `fibonacci(4)` returns 3\n\nFinally, `fibonacci(6)` returns 5 + 3 = 8.', stop_reason=None)], created=1723743390, model='hermes3-405b', object='text_completion', system_fingerprint=None, usage=CompletionUsage(completion_tokens=1026, prompt_tokens=4, total_tokens=1030))
    ```

## Listing models

The `/models` endpoint lists the models available for use through the Lambda
Chat API.

To use the `/models` endpoint:

=== "Curl"

    Run:

    ```bash
    curl https://api.lambdalabs.com/v1/models -H "Authorization: Bearer <API-KEY>" -H "Content-Type: application/json" | jq .
    ```

    You should see output similar to:

    ```json
    {
      "id": "hermes3-70b",
      "object": "model",
      "created": 1724347380,
      "owned_by": "lambda"
    },
    {
      "id": "hermes3-8b",
      "object": "model",
      "created": 1724347380,
      "owned_by": "lambda"
    },
    {
      "id": "lfm-40b",
      "object": "model",
      "created": 1724347380,
      "owned_by": "lambda"
    },
    {
      "id": "llama3.1-405b-instruct-fp8",
      "object": "model",
      "created": 1724347380,
      "owned_by": "lambda"
    },
    ```

=== "Python"

    First,
    [create and activate a Python virtual environment](../education/programming/virtual-environments-containers.md#creating-a-python-virtual-environment).
    Then, install the
    [OpenAI Python API library](https://pypi.org/project/openai/){ .external target="_blank" }.

    Run:

    ```python
    from openai import OpenAI

    openai_api_key = "<API-KEY>"
    openai_api_base = "https://api.lambdalabs.com/v1"

    client = OpenAI(
        api_key=openai_api_key,
        base_url=openai_api_base,
    )

    client.models.list()
    ```

    You should see output similar to:

    ```
    SyncPage[Model](data=[Model(id='hermes3-405b', created=1677610602, object='model', owned_by='openai'), Model(id='hermes-3-llama-3.1-405b-fp8-128k', created=1677610602, object='model', owned_by='openai')], object='list')
    ```
