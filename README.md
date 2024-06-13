# LLM chat templates

This is a collection of Jinja2 chat templates for LLMs, for both text and vision (text + image inputs) models. 

Many of these templates originated from the ones included in the [Sibila project](https://github.com/jndiogo/sibila).


All the templates can be applied by the following code:

``` python
from jinja2.sandbox import ImmutableSandboxedEnvironment

jinja_env = ImmutableSandboxedEnvironment(trim_blocks=True,
                                          lstrip_blocks=True)

format_template = """..."""

jinja_compiled_template = jinja_env.from_string(format_template)


messages = [
    {'role': 'system',  'content': 'You speak like a pirate.'},
    {'role': 'user', 'content': 'Can you teach me to swim?'},
    {'role': 'assistant', 'content': 'Sure thing, matey! Then ye go swim with the shark.'}
]

text = jinja_compiled_template.render(messages=messages,
                                      add_generation_prompt=True,
                                      **{"bos_token": "...",
                                         "eos_token": "..."})
```

Some models were not trained with support for system/instructions message. For these models, the template will prepend such message to the first user message. The system message (if any) should always be the first message in the thread.

The templates deal with add_generation_prompt and therefore terminate by adding the header of the next assistant message, so that the model continues from there, except when the model does not need it.

Most entries list the HuggingFace link for the model, usually to a GGUF version.

These chat templates were compiled from many sources, including:
- [HuggingFace](https://huggingface.co/) models' tokenizer_config.json files
- https://github.com/chujiezheng/chat_templates/tree/main

Some were adapted to deal with system messages and "add_generation_prompt".

Most of the templates were tested, but if you notice an error, please open an issue!





## Text Models

- [ChatML](#chatml)
- [Llama-3 Instruct](#llama-3-instruct)
- [Mistral](#mistral)
- [OpenChat 3.5](#openchat-35)
- [OpenChat 3.5 Gemma](#openchat-35-gemma)
- [OpenChat 3.6](#openchat-36)
- [Phi-2](#phi-2)
- [Phi-3](#phi-3)
- [Vicuna](#vicuna)
- [Zephyr](#zephyr)
- [Zephyr-gemma](#zephyr-gemma)


## Vision Models (text + image input)

- [LLaVA 1.5](#llava-15)
- [LLaVA 1.6 Mistral](#llava-16-mistral)
- [LLaVA 1.6 Vicuna](#llava-16-vicuna)
- [LLaVA 1.6 Hermes](#llava-16-hermes)
- [Moondream2](#moondream2)
- [LLaVA-phi3](#llava-phi3)
- [LLaVA-Llama-3](#llava-llama-3)
- [Llama-3-vision](#llama-3-vision)






## Alpaca

https://huggingface.co/TheBloke/claude2-alpaca-13B-GGUF

```
{% if messages[0]['role'] == 'system' %}
  {% set loop_messages = messages[1:] %}
  {% set system_message = messages[0]['content'].strip() + '\n' %}
{% else %}
  {% set loop_messages = messages %}
  {% set system_message = '' %}
{% endif %}

{% for message in loop_messages %}

  {% if (message['role'] == 'user') != (loop.index0 % 2 == 0) %}
    {{ raise_exception('Conversation roles must alternate user/assistant/user/assistant/...') }}
  {% endif %}
  
  {% if loop.index0 == 0 %}
    {% set content = system_message + message['content'] %}
  {% else %}
    {% set content = message['content'] %}
  {% endif %}
  
  {% if message['role'] == 'user' %}
    {{ '### Instruction:\n' + content.strip() + '\n\n'}}
  {% elif message['role'] == 'assistant' %}
    {{ '### Response:\n'  + content.strip() + '\n\n' }}
  {% endif %}
  
{% endfor %}

{% if add_generation_prompt %}
  {{ '### Response:\n' }}
{% endif %}
``` 


## AmberChat

https://huggingface.co/TheBloke/AmberChat-GGUF

```
{% if messages[0]['role'] == 'system' %}
  {% set loop_messages = messages[1:] %}
  {% set system_message = messages[0]['content'].strip() + '\n' %}
{% else %}
  {% set loop_messages = messages %}
  {% set system_message = '' %}
{% endif %}

{% for message in loop_messages %}
  {% if (message['role'] == 'user') != (loop.index0 % 2 == 0) %}
    {{ raise_exception('Conversation roles must alternate user/assistant/user/assistant/...') }}
  {% endif %}
  
  {% if loop.index0 == 0 %}
    {{ bos_token + system_message }}
  {% endif %}
  {% if message['role'] == 'user' %}
    {{ '### Human: ' + message['content'].strip() + '\n' }}
  {% elif message['role'] == 'assistant' %}
    {{ '### Assistant: ' + message['content'].strip() + '\n' }}
  {% endif %}
{% endfor %}

{% if add_generation_prompt %}
  {{ '### Assistant:' }}
{% endif %}
```

## Command-R

https://huggingface.co/andrewcanis/c4ai-command-r-v01-GGUF

```
{{ bos_token }}
{% if messages[0]['role'] == 'system' %}
  {% set loop_messages = messages[1:] %}
  {% set system_message = messages[0]['content'] %}
{% else %}
  {% set loop_messages = messages %}
  {% set system_message = false %}
{% endif %}

{% if system_message != false %}
  {{ '<|START_OF_TURN_TOKEN|><|SYSTEM_TOKEN|>' + system_message + '<|END_OF_TURN_TOKEN|>' }}
{% endif %}

{% for message in loop_messages %}
  {% if (message['role'] == 'user') != (loop.index0 % 2 == 0) %}
    {{ raise_exception('Conversation roles must alternate user/assistant...') }}
  {% endif %}
  
  {% set content = message['content'] %}
  {% if message['role'] == 'user' %}
    {{ '<|START_OF_TURN_TOKEN|><|USER_TOKEN|>' + content.strip() + '<|END_OF_TURN_TOKEN|>' }}
  {% elif message['role'] == 'assistant' %}
    {{ '<|START_OF_TURN_TOKEN|><|CHATBOT_TOKEN|>'  + content.strip() + '<|END_OF_TURN_TOKEN|>' }}
  {% endif %}
{% endfor %}

{% if add_generation_prompt %}
  {{ '<|START_OF_TURN_TOKEN|><|CHATBOT_TOKEN|>' }}
{% endif %}
```



## ChatML

```
{% for message in messages %}
    {{'<|im_start|>' + message['role'] + '\n' + message['content'].strip() + '<|im_end|>' + '\n'}}
{% endfor %}

{% if add_generation_prompt %}
    {{ '<|im_start|>assistant\n' }}
{% endif %}
```




## Llama-3 Instruct

https://huggingface.co/bartowski/Meta-Llama-3-8B-Instruct-GGUF


```
{{ bos_token }}
{% if messages[0]['role'] == 'system' %}
  {% set loop_messages = messages[1:] %}
  {% set system_message = '<|start_header_id|>' + 'system' + '<|end_header_id|>\n\n' + messages[0]['content'].strip() + '<|eot_id|>' %}
{% else %}
  {% set loop_messages = messages %}
  {% set system_message = '' %}
{% endif %}

{% for message in loop_messages %}

  {% if (message['role'] == 'user') != (loop.index0 % 2 == 0) %}
    {{ raise_exception('Conversation roles must alternate user/assistant/user/assistant/...') }}
  {% endif %}

  {% if loop.index0 == 0 %}
    {{ system_message }}
  {% endif %}
  
  {{ '<|start_header_id|>' + message['role'] + '<|end_header_id|>\n\n' + message['content'].strip() + '<|eot_id|>' }}

  {% if loop.last and message['role'] == 'user' and add_generation_prompt %}
    {{ '<|start_header_id|>' + 'assistant' + '<|end_header_id|>\n\n' }}
  {% endif %}

{% endfor %}
```


## Mistral

https://huggingface.co/mistralai/Mistral-7B-Instruct-v0.3

https://huggingface.co/MaziyarPanahi/Mistral-7B-Instruct-v0.3-GGUF

```
{{ bos_token }}
{% if messages[0]['role'] == 'system' %}
  {% set loop_messages = messages[1:] %}
  {% set system_message = messages[0]['content'].strip() + '\n\n' %}
{% else %}
  {% set loop_messages = messages %}
  {% set system_message = '' %}
{% endif %}

{% for message in loop_messages %}

  {% if (message['role'] == 'user') != (loop.index0 % 2 == 0) %}
    {{ raise_exception('Conversation roles must alternate user/assistant/user/assistant/...') }}
  {% endif %}

  {% if loop.index0 == 0 %}
    {% set content = system_message + message['content'] %}
  {% else %}
    {% set content = message['content'] %}
  {% endif %}
  
  {% if message['role'] == 'user' %}
    {{ '[INST] ' + content.strip() + ' [/INST]' }}
  {% elif message['role'] == 'assistant' %}
    {{ content.strip() + eos_token}}
  {% endif %}

{% endfor %}"
```




## OpenChat 3.5

https://huggingface.co/TheBloke/openchat_3.5-GGUF

```
{{ bos_token }}
{% for message in messages %}
  {{ 'GPT4 Correct ' + message['role'].title() + ': ' + message['content'].strip() + '<|end_of_turn|>'}}
{% endfor %}

{% if add_generation_prompt %}
  {{ 'GPT4 Correct Assistant:' }}
{% endif %}
```


## OpenChat 3.5 Gemma

https://huggingface.co/bartowski/openchat-3.5-0106-gemma-GGUF

```
{{ bos_token }}
{% if messages[0]['role'] == 'system' %}
  {% set loop_messages = messages[1:] %}
  {% set system_message = messages[0]['content'].strip() + '\n\n' %}
{% else %}
  {% set loop_messages = messages %}
  {% set system_message = '' %}
{% endif %}

{% for message in loop_messages %}
  {% if (message['role'] == 'user') != (loop.index0 % 2 == 0) %}
    {{ raise_exception('Conversation roles must alternate user/assistant/user/assistant/...') }}
  {% endif %}  
  {% if loop.index0 == 0 %}
    {% set content = system_message + message['content'] %}
  {% else %}
    {% set content = message['content'] %}
  {% endif %}
  
  {{ 'GPT4 Correct ' + message['role'].title() + ': ' + content + '<end_of_turn>'}}
{% endfor %}

{% if add_generation_prompt %}
  {{ 'GPT4 Correct Assistant:' }}
{% endif %}
```

## OpenChat 3.6

https://huggingface.co/bartowski/openchat-3.6-8b-20240522-GGUF

```
{{ bos_token }}
{% for message in messages %}
  {% if message['role'] in ['user', 'assistant'] %}
    {% set content = '<|start_header_id|>GPT4 Correct ' + message['role'].title() + '<|end_header_id|>\n\n' + message['content'] | trim + '<|eot_id|>' %}
  {% elif message['role'] == 'system' %}
    {% set content = '<|start_header_id|>System<|end_header_id|>\n\n' + message['content'] | trim + '<|eot_id|>' %}
  {% else %}
    {{ raise_exception('Only user, assistant and system roles are supported!') }}
  {% endif %}
  
  {{ content }}
{% endfor %}

{% if add_generation_prompt %}
  {{ '<|start_header_id|>GPT4 Correct Assistant<|end_header_id|>\n\n' }}
{% endif %}
```


## Phi-2

https://huggingface.co/TheBloke/phi-2-GGUF

```
{% if messages[0]['role'] == 'system' %}
  {% set loop_messages = messages[1:] %}
  {% set system_message = messages[0]['content'].strip() + '\n\n' %}
{% else %}
  {% set loop_messages = messages %}
  {% set system_message = '' %}
{% endif %}

{% for message in loop_messages %}
  {% if (message['role'] == 'user') != (loop.index0 % 2 == 0) %}
    {{ raise_exception('Conversation roles must alternate user/assistant/user/assistant/...') }}
  {% endif %}
  
  {% if loop.index0 == 0 %}
    {% set content = system_message + message['content'] %}
  {% else %}
    {% set content = message['content'] %}
  {% endif %}
  
  {% if message['role'] == 'user' %}
    {{ 'Instruct: ' + content.strip() + '\n' }}
  {% elif message['role'] == 'assistant' %}
    {{ 'Output: '  + content.strip() + '\n' }}
  {% endif %}
{% endfor %}

{% if add_generation_prompt %}
  {{ 'Output:' }}
{% endif %}
```


## Phi-3

https://huggingface.co/microsoft/Phi-3-mini-4k-instruct


```
{{ bos_token }}
{% for message in messages %}
    {% if (message['role'] == 'system') %}
        {{'<|system|>' + '\n' + message['content'].strip() + '<|end|>' + '\n'}}
    {% elif (message['role'] == 'user') %}
        {{'<|user|>' + '\n' + message['content'].strip() + '<|end|>' + '\n' + '<|assistant|>' + '\n'}}
    {% elif message['role'] == 'assistant' %}
        {{message['content'].strip() + '<|end|>' + '\n'}}
    {% endif %}
{% endfor %}"
```





## Vicuna

https://huggingface.co/lmsys/vicuna-7b-v1.5

https://huggingface.co/TheBloke/vicuna-7B-v1.5-GGUF

```
{% if messages[0]['role'] == 'system' %}
  {% set loop_messages = messages[1:] %}
  {% set system_message = messages[0]['content'].strip() + '\n\n' %}
{% else %}
  {% set loop_messages = messages %}
  {% set system_message = '' %}
{% endif %}

{{ system_message }}

{% for message in loop_messages %}
  {% if (message['role'] == 'user') != (loop.index0 % 2 == 0) %}
    {{ raise_exception('Conversation roles must alternate user/assistant/user/assistant/...') }}
  {% endif %}
  
  {% if message['role'] == 'user' %}
    {{ 'USER: ' + message['content'].strip() + '\n' }}
  
  {% elif message['role'] == 'assistant' %}
  
    {{ 'ASSISTANT: ' + message['content'].strip() + eos_token + '\n' }}
  {% endif %}
  
{% endfor %}

{% if add_generation_prompt %}
  {{ 'ASSISTANT:' }}
{% endif %}
```


## Zephyr

https://huggingface.co/TheBloke/zephyr-7B-beta-GGUF

```
{% for message in messages %}

  {% if message['role'] == 'user' %}
    {{ '<|user|>\n' + message['content'] + eos_token }}
  {% elif message['role'] == 'system' %}
    {{ '<|system|>\n' + message['content'] + eos_token }}
  {% elif message['role'] == 'assistant' %}
    {{ '<|assistant|>\n'  + message['content'] + eos_token }}
  {% endif %}
  
  {% if loop.last and add_generation_prompt %}
    {{ '<|assistant|>' }}
  {% endif %}

{% endfor %}
```




## Zephyr-gemma

https://huggingface.co/LoneStriker/zephyr-7b-gemma-v0.1-GGUF

```
{% if messages[0]['role'] == 'user' or messages[0]['role'] == 'system' %}
  {{ bos_token }}
{% endif %}

{% for message in messages %}
  {{ '<|im_start|>' + message['role'] + '\n' + message['content'] + '<|im_end|>' + '\n' }}
{% endfor %}

{% if add_generation_prompt %}
  {{ '<|im_start|>assistant\n' }}
{% elif messages[-1]['role'] == 'assistant' %}
  {{ eos_token }}
{% endif %}"
```




# Vision Models (text + image input)


## LLaVA 1.5

https://huggingface.co/mys/ggml_llava-v1.5-7b


```
{% for message in messages %}
  {% if message.role == 'system' %}
    {{ message.content.strip() }}
    {{ '\n' }}

  {% elif message.role == 'user' %}
    {% if message.content is string %}
      USER: {{ message.content.strip() }}

    {% elif message.content is iterable %}
      USER: 
      
      {% for content in message.content %}
        
        {% if content.type == 'image_url' and content.image_url is mapping and content.image_url.url is string%}
          {{ content.image_url.url.strip() + ' ' }}
        {% endif %}

      {% endfor %}

      {% for content in message.content %}

        {% if content.type == 'text' %}
          {{ content.text.strip() }}
        {% endif %}

      {% endfor %}

    {% endif %}

    {{ '\n' }}

  {% elif message.role == 'assistant' and message.content is not none %}
    ASSISTANT: {{ message.content.strip() }}
    {{ '\n' }}
  {% endif %}

{% endfor %}

{% if add_generation_prompt %}
  ASSISTANT: 
{% endif %}
```




## LLaVA 1.6 Mistral

https://huggingface.co/cjpais/llava-1.6-mistral-7b-gguf


```
{% if messages[0]['role'] == 'system' %}
  {% set loop_messages = messages[1:] %}
  {% set system_message = messages[0]['content'].strip() + ' ' %}
{% else %}
  {% set loop_messages = messages %}
  {% set system_message = '' %}
{% endif %}

{% for message in loop_messages %}

  {% if (message['role'] == 'user') != (loop.index0 % 2 == 0) %}
    {{ raise_exception('Conversation roles must alternate user/assistant/user/assistant/...') }}
  {% endif %}

  {% if message.role == 'user' %}

    [INST]
  
    {% if message.content is string %}

      {% if system_message != '' %}
        {% set text = system_message + message.content.strip() %}
        {% set system_message = '' %}
      {% else %}
        {% set text = message.content.strip() %}
      {% endif %}

      {{ text }}

    {% elif message.content is iterable %}

      {% for content in message.content %}
        {% if content.type == 'image_url' and content.image_url is mapping %}
          {{ content.image_url.url.strip() + '\n' }}
        {% endif %}
      {% endfor %}

      {% for content in message.content %}
        {% if content.type == 'text' %}

          {% if system_message != '' %}
            {% set text = system_message + content.text.strip() %}
            {% set system_message = '' %}
          {% else %}
            {% set text = content.text.strip() %}
          {% endif %}

          {{ text }}
        {% endif %}
      {% endfor %}

    {% endif %}

    [/INST]

  {% elif message.role == 'assistant' %}
    {{ message.content.strip() }}
  {% endif %}

{% endfor %}

{% if add_generation_prompt %}
{% endif %}
```



## LLaVA 1.6 Vicuna

https://huggingface.co/cjpais/llava-v1.6-vicuna-7b-gguf


```
{% if messages[0]['role'] == 'system' %}
  {% set loop_messages = messages[1:] %}
  {% set system_message = messages[0]['content'].strip() + ' ' %}
{% else %}
  {% set loop_messages = messages %}
  {% set system_message = '' %}
{% endif %}

{% for message in loop_messages %}

  {% if message.role == 'user' %}
    USER:{{ '\n' }}
  
    {% if message.content is string %}

      {% if system_message != '' %}
        {% set text = system_message + message.content.strip() %}
        {% set system_message = '' %}
      {% else %}
        {% set text = message.content.strip() %}
      {% endif %}

      {{ text }}

    {% elif message.content is iterable %}

      {% for content in message.content %}
        {% if content.type == 'image_url' and content.image_url is mapping %}
          {{ content.image_url.url + ' ' }}
        {% endif %}
      {% endfor %}

      {% for content in message.content %}
        {% if content.type == 'text' %}

          {% if system_message != '' %}
            {% set text = system_message + content.text.strip() %}
            {% set system_message = '' %}
          {% else %}
            {% set text = content.text.strip() %}
          {% endif %}

          {{ text }}
        {% endif %}
      {% endfor %}

    {% endif %}

    {{ '\n' }}

  {% elif message.role == 'assistant' %}
    ASSISTANT:{{ '\n' + message.content + '\n' }}
  {% endif %}

{% endfor %}

{% if add_generation_prompt %}
  ASSISTANT:{{ '\n' }}
{% endif %}
```



## LLaVA 1.6 Hermes

Based on Nous-Hermes-2-Yi-34B.

https://huggingface.co/cjpais/llava-v1.6-34B-gguf


```
{% for message in messages %}

  {% if message.role == 'user' %}
    {{ '<|im_start|>user\n' }}

    {% if message.content is string %}
        {{ message.content.strip() }}

    {% elif message.content is iterable %}
      
      {% for content in message.content %}
        
        {% if content.type == 'image_url' and content.image_url is mapping and content.image_url.url is string%}
          {{ content.image_url.url.strip() + '\n' }}
        {% endif %}

      {% endfor %}

      {% for content in message.content %}

        {% if content.type == 'text' %}
          {{ content.text.strip() }}
        {% endif %}

      {% endfor %}

    {% endif %}

    {{ '<|im_end|>' + '\n' }}

  {% elif message.role == 'assistant' or message.role == 'system' and message.content is not none %}
    {{ '<|im_start|>' + message.role + '\n' + message.content.strip() + '<|im_end|>' + '\n' }}
  {% endif %}

{% endfor %}

{% if add_generation_prompt %}
    {{ '<|im_start|>assistant\n' }}
{% endif %}
```







## Moondream2

https://huggingface.co/vikhyatk/moondream2


```
{% if messages[0]['role'] == 'system' %}
  {% set loop_messages = messages[1:] %}
  {% set system_message = messages[0]['content'].strip() + '\n' %}
{% else %}
  {% set loop_messages = messages %}
  {% set system_message = '' %}
{% endif %}

{% for message in loop_messages %}

  {% if message.role == 'user' %}
  
    {% if message.content is string %}

      {% if system_message != '' %}
        {% set text = system_message + message.content.strip() %}
        {% set system_message = '' %}
      {% else %}
        {% set text = message.content.strip() %}
      {% endif %}

      Question: {{ text + '\n\n' }}

    {% elif message.content is iterable %}

      {% for content in message.content %}
        {% if content.type == 'image_url' and content.image_url is mapping %}
          {{ content.image_url.url + '\n\n' }}
        {% endif %}
      {% endfor %}

      {% for content in message.content %}
        {% if content.type == 'text' %}

          {% if system_message != '' %}
            {% set text = system_message + content.text.strip() %}
            {% set system_message = '' %}
          {% else %}
            {% set text = content.text.strip() %}
          {% endif %}

          Question: {{ text + '\n\n' }}
        {% endif %}
      {% endfor %}

    {% endif %}

  {% elif message.role == 'assistant' %}
    Answer: {{ message.content + '\n\n' }}
  {% endif %}

{% endfor %}

{% if add_generation_prompt %}
  Answer: 
{% endif %}
```


## LLava-phi3

https://huggingface.co/xtuner/llava-phi-3-mini-gguf


{{ bos_token }}
{% for message in messages %}
    {% if (message['role'] == 'system') %}
        {{'<|system|>' + '\n' + message['content'].strip() + '<|end|>' + '\n'}}
    {% elif (message['role'] == 'user') %}
        {{'<|user|>' + '\n' + message['content'].strip() + '<|end|>' + '\n' + '<|assistant|>' + '\n'}}
    {% elif message['role'] == 'assistant' %}
        {{message['content'].strip() + '<|end|>' + '\n'}}
    {% endif %}
{% endfor %}"

-------------------------

```
{{ bos_token }}
{% for message in messages %}
  {% if message.role == 'system' %}
    {{'<|system|>' + '\n' + message.content.strip() + '<|end|>' + '\n'}}

  {% elif message.role == 'user' %}
    {{ '<|user|>' + '\n' }}

    {% if message.content is string %}
      {{ message.content.strip() }}

    {% elif message.content is iterable %}
      
      {% for content in message.content %}
        
        {% if content.type == 'image_url' and content.image_url is mapping and content.image_url.url is string%}
          {{ content.image_url.url.strip() + '\n' }}
        {% endif %}

      {% endfor %}

      {% for content in message.content %}

        {% if content.type == 'text' %}
          {{ content.text.strip() }}
        {% endif %}

      {% endfor %}

    {% endif %}

    {{ '<|end|>' + '\n' + '<|assistant|>' }}

  {% elif message.role == 'assistant' and message.content is not none %}
    {{ message['content'].strip() + '<|end|>' + '\n' }}

  {% endif %}

{% endfor %}
```








## Llava-Llama-3

https://huggingface.co/xtuner/llava-llama-3-8b-v1_1-gguf


```
{{ bos_token }}
{% if messages[0]['role'] == 'system' %}
  {% set loop_messages = messages[1:] %}
  {% set system_message = '<|start_header_id|>' + 'system' + '<|end_header_id|>\n\n' + messages[0]['content'].strip() + '<|eot_id|>' %}
{% else %}
  {% set loop_messages = messages %}
  {% set system_message = '' %}
{% endif %}

{% for message in loop_messages %}

  {% if (message['role'] == 'user') != (loop.index0 % 2 == 0) %}
    {{ raise_exception('Conversation roles must alternate user/assistant/user/assistant/...') }}
  {% endif %}

  {% if loop.index0 == 0 %}
    {{ system_message }}
  {% endif %}
  
  {{ '<|start_header_id|>' + message['role'] + '<|end_header_id|>\n\n' }}
  
  {% if message.content is string %}
    {{ message.content.strip() }}

  {% elif message.content is iterable %}
    
    {% for content in message.content %}
      
      {% if content.type == 'image_url' and content.image_url is mapping and content.image_url.url is string%}
        {{ content.image_url.url + '\n' }}
      {% endif %}

    {% endfor %}

    {% for content in message.content %}

      {% if content.type == 'text' %}
        {{ content.text.strip() }}
      {% endif %}

    {% endfor %}

  {% endif %}

  {{ '<|eot_id|>' }}

  {% if loop.last and message['role'] == 'user' and add_generation_prompt %}
    {{ '<|start_header_id|>' + 'assistant' + '<|end_header_id|>\n\n' }}
  {% endif %}

{% endfor %}
```



## Llama-3-vision

https://huggingface.co/qresearch/llama-3-vision-alpha-hf


```
{{ bos_token }}
{% if messages[0]['role'] == 'system' %}
  {% set loop_messages = messages[1:] %}
  {% set system_message = '<|start_header_id|>' + 'system' + '<|end_header_id|>\n\n' + messages[0]['content'].strip() + '<|eot_id|>' %}
{% else %}
  {% set loop_messages = messages %}
  {% set system_message = '' %}
{% endif %}

{% for message in loop_messages %}

  {% if (message['role'] == 'user') != (loop.index0 % 2 == 0) %}
    {{ raise_exception('Conversation roles must alternate user/assistant/user/assistant/...') }}
  {% endif %}

  {% if loop.index0 == 0 %}
    {{ system_message }}
  {% endif %}
  
  {{ '<|start_header_id|>' + message['role'] + '<|end_header_id|>\n\n' }}
  
  {% if message.content is string %}
    {{ message.content.strip() }}

  {% elif message.content is iterable %}
    
    {% for content in message.content %}
      
      {% if content.type == 'image_url' and content.image_url is mapping and content.image_url.url is string%}
        {{ content.image_url.url + '\n' }}
      {% endif %}

    {% endfor %}

    {% for content in message.content %}

      {% if content.type == 'text' %}
        {{ content.text.strip() }}
      {% endif %}

    {% endfor %}

  {% endif %}

  {{ '<|eot_id|>' }}

  {% if loop.last and message['role'] == 'user' and add_generation_prompt %}
    {{ '<|start_header_id|>' + 'assistant' + '<|end_header_id|>\n\n' }}
  {% endif %}

{% endfor %}
```


