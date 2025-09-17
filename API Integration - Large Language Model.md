# API Reference for Large Language Model (LLM)

* Kramdown TOC here
{:toc}

## Using LiteLLM

### Installation

```bash
pip install litellm
```

### Basic Usage with Streaming
#### Async Streaming Completion
The AI Marketplace API uses API keys for authentication. Please contact our team to acquire your API Key. 

```python
from litellm import acompletion
import json
import asyncio

async def stream_response():
    try:
        # Initialize the completion request
        response = await acompletion(
            model="{model-name}", 
            api_base="https://mkp-api.fptcloud.com",    # Base URL for API
            api_key="{api-key}",          # Your API key
            messages=[                    # List of message objects. Please update the System prompt to have the model respond appropriately
                {
                    "role": "system",
                    "content": "You are a helpful assistant capable of understanding a user's needs through conversation to recommend suitable services. Based on the conversation history and the user's last message, list services that can address the user's needs. Respond only in Vietnamese or English, matching the language of the user's input."
                },
                {
                    "role": "user",
                    "content": "{your-input-text}"
                }
            ],
            stream=True  # Enable streaming
        )
        # Process streaming response
        async for chunk in response:
            if chunk.choices[0].delta.content:
                yield f"data: {json.dumps({'content': chunk.choices[0].delta.content})}\n\n"
    except Exception as e:
        yield f"data: {json.dumps({'error': str(e)})}\n\n"
    yield "data: [DONE]\n\n"

async def main():
    async for data in stream_response():
        print(data)

if __name__ == '__main__':
    asyncio.run(main())
```

## Python

```python
import requests
import json

url = "https://mkp-api.fptcloud.com/chat/completions"

token = ""
headers = {
    "Content-Type": "application/json",
    "Authorization": f"Bearer {api-key}"
}

data = {
    "model": "{model-name}",                           # Model name

    "messages": [                                      # List of message objects. Please update the System prompt to have the model respond appropriately
        {
            "role": "system",
            "content": "You are a helpful assistant capable of understanding a user's needs through conversation to recommend suitable services. Based on the conversation history and the user's last message, list services that can address the user's needs. Respond only in Vietnamese or English, matching the language of the user's input."
        },
        {
            "role": "user",
            "content": "{your-input-text}"
        }
    ],
    "stream": True                                      # Enable streaming
}

# Since stream=True, we need to handle streaming response
response = requests.post(url, headers=headers, data=json.dumps(data), stream=True)

# Process the streaming response
for line in response.iter_lines():
    if line:
        # Skip the "data: " prefix if present
        line_text = line.decode('utf-8')
        if line_text.startswith('data: '):
            line_text = line_text[6:]
        
        # Skip empty lines or "[DONE]" message
        if line_text == "[DONE]":
            break
        
        try:
            # Parse the JSON response chunk
            json_response = json.loads(line_text)
            # Process the chunk as needed
            print(json_response)
        except json.JSONDecodeError:
            # Handle non-JSON lines
            print(f"Cannot parse: {line_text}")
```

## cURL

```shell
curl --location 'https://mkp-api.fptcloud.com/chat/completions' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer {api-key}' \
--header 'Cookie: cf_use_ob=0' \
--data '{
 
    "model": "{model-name}",
 
    "messages": [
 
        {
      "role": "system",
      "content": "You are a helpful assistant capable of understanding a user'\''s needs through conversation to recommend suitable services. Based on the conversation history and the user'\''s last message, list services that can address the user'\''s needs. Respond only in Vietnamese or English, matching the language of the user'\''s input."
    },
    {
 
            "role": "user",
 
            "content": "hi"
 
        }
    ],
    "stream": false
 
}'   
 
```

## Langchain

```python
#!/usr/bin/env python3
import os
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate, SystemMessagePromptTemplate, HumanMessagePromptTemplate
from langchain_core.output_parsers import StrOutputParser

API_KEY = "s"
BASE_URL = "https://mkp-api.fptcloud.com"
MODEL = ""

# Check if API_KEY is set (optional but good practice)
if not API_KEY:
    # Attempt to get from environment variable, or raise error/warning
    API_KEY = os.getenv("FPT_API_KEY") # Example environment variable name
    if not API_KEY:
        print("Warning: API_KEY is not set. Please set the API_KEY variable or FPT_API_KEY environment variable.")
        exit()

if not MODEL:
     print("Warning: MODEL name is not set. Please set the MODEL variable.")
     exit()


# Chat Model - Configure ChatOpenAI to use your custom endpoint
chat_model = ChatOpenAI(
    model=MODEL,
    openai_api_key=API_KEY,
    openai_api_base=BASE_URL,
    temperature=0.0,
)

# Prompt Template - Define system and user messages
system_prompt_template = SystemMessagePromptTemplate.from_template(
    "You are a faithful Vietnamese assistant. Provide detailed answers. Use bullet points if necessary. Provide your answer in Vietnamese. Do not include any other text or instructions. Only provide the answer."
)
human_prompt_template = HumanMessagePromptTemplate.from_template("{user_prompt}") 

chat_prompt = ChatPromptTemplate.from_messages(
    [system_prompt_template, human_prompt_template]
)

# Output Parser - To get a simple string output
output_parser = StrOutputParser()

# Chains the prompt formatting, model invocation, and output parsing
chain = chat_prompt | chat_model | output_parser

def chat_stream_langchain(prompt: str):
    """Chats with the model using Langchain and streams the response."""
    print("Streaming response:")

    try:
        for chunk in chain.stream({"user_prompt": prompt}):
            print(chunk, end='', flush=True)
        print("\n--- End of Stream ---")
    except Exception as e:
        print(f"\nAn error occurred during streaming: {e}")

def chat_non_stream_langchain(prompt: str):
    """Chats with the model using Langchain without streaming."""
    print("Getting non-streaming response...")
    
    try:
        response = chain.invoke({"user_prompt": prompt})
        print("\nResponse:")
        print(response)
        print("--- End of Response ---")
    except Exception as e:
        print(f"\nAn error occurred during invocation: {e}")

vietnamese_prompt = "Bạn có thể giúp tôi mô tả về hệ mặt trời không?"

# print("\nTesting Non-Streaming Function:")
# chat_non_stream_langchain(vietnamese_prompt)

print("\nTesting Streaming Function:")
chat_stream_langchain(vietnamese_prompt)
```

## OpenAI

```python
#!/usr/bin/env python3
import base64
import requests
import io
from io import BytesIO
from PIL import Image
from openai import OpenAI

API_KEY = ""
BASE_URL = "https://mkp-api.fptcloud.com"
MODEL=""

client = OpenAI(
    api_key=API_KEY,
    base_url=BASE_URL
)

def chat_stream(prompt: str):
                              
    chat_completion = client.chat.completions.create(
        messages=[
            {
                "role": "system",
                "content": "You are a faithful Vietnamese assistant. Provide detailed answers. Use bullet points if necessary. Provide your answer in Vietnamese. Do not include any other text or instructions. Only provide the answer.",
            },
            {
                "role": "user",
                "content": f"{prompt}"
            },
        ],
        model=MODEL,
        temperature=0.0,
        stream=True,  # this time, we set stream=True,
    )

    for chunk in chat_completion:
        if chunk is not None and chunk.choices[0].delta.content is not None:
            print(chunk.choices[0].delta.content, end='', flush=True)
    print("")

def chat_non_stream(prompt: str):
                              
    chat_completion = client.chat.completions.create(
        messages=[
            {
                "role": "system",
                "content": "You are a faithful Vietnamese assistant. Provide detailed answers. Use bullet points if necessary. Provide your answer in Vietnamese. Do not include any other text or instructions. Only provide the answer.",
            },
            {
                "role": "user",
                "content": f"{prompt}"
            },
        ],
        model=MODEL,
        temperature=0.0
    )
    print(chat_completion.choices[0].message.content)
chat_non_stream("Bạn có thể giúp tôi mô tả về hệ mặt trời không?")
```

## Nodejs

```python
const OpenAI = require('openai');
const API_KEY = "";
const BASE_URL = "https://mkp-api.fptcloud.com";
const MODEL = "SaoLa-Llama3.1-planner"; 
const USER_PROMPT = "Xin chào" 
const openai = new OpenAI({ apiKey: API_KEY, baseURL: BASE_URL});
async function getLLMResponseWithOpenAI(prompt, model) {
    try {
        const response = await openai.chat.completions.create({
            model: model,
            messages: [{ role: "user", content: prompt }],
        });

        if (response.choices && response.choices.length > 0 && response.choices[0].message && response.choices[0].message.content) {
            return response.choices[0].message.content;
        } else {
            console.error("Failed to retrieve response from OpenAI.");
            return null;
        }
    } catch (error) {
        console.error("Error retrieving response from OpenAI:", error);
        return null;
    }
}
async function main() {
    const llmResponse = await getLLMResponseWithOpenAI(USER_PROMPT, MODEL);

    if (llmResponse) {
        console.log("\nLLM Response received:");
        console.log(llmResponse);
    } else {
        console.log("\nFailed to retrieve a response from the LLM using the OpenAI library.");
    }
}
main();
```

## Key Parameters Explained
- `model`: String identifier for the model you want to use
- `api_base`: The base URL endpoint for your API
- `api_key`: Your authentication API key
- `messages`: List of message objects with the following structure:
  - `role`: Can be "system", "user", or "assistant"
  - `content`: The actual message content
- `stream`: Boolean flag to enable streaming (set to `True` for streaming responses)

## Response Format
The streaming response will yield chunks in the following format:

```json
data: {"content": "chunk-of-response-text"}
```

If an error occurs:

```json
data: {"error": "error-message"}
```

End of stream marker:

```
data: [DONE]
```

## Error Handling
The code includes try-catch block to handle potential errors during the API call and streaming process. Any errors will be returned as JSON-encoded error messages in the stream.

## Notes
- Make sure to handle the async nature of the function with appropriate async/await syntax
- The streaming response is formatted as Server-Sent Events (SSE)
- Each chunk is JSON-encoded and prefixed with "data: "
- The stream ends with a [DONE] marker




