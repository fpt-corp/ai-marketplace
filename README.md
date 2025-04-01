# Using LiteLLM

## Installation
```bash
pip install litellm
```
## Basic Usage with Streaming
### Async Streaming Completion
The AI Marketplace API uses API keys for authentication. Please contact our team to acquire your API Key. 

```python
from litellm import acompletion
import json
import asyncio

async def stream_response():
    try:
        # Initialize the completion request
        response = await acompletion(
            model="openai/{model-name}",  # Prefix required openai/
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
# Python
```python
import requests
import json

url = "https://mkp-api.fptcloud.com/chat/completions"

token = ""
headers = {
    "Content-Type": "application/json",
    "Authorization": f"Bearer {token}"
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
# cURL
```cURL
curl --location "https://mkp-api.fptcloud.com/chat/completions" \
--header "Content-Type: application/json" \
--header "Authorization: Bearer YOUR_API_KEY" \
--data '{
    "model": "YOUR_MODEL_NAME",
    "messages": [
      {
        "role": "system",
        "content": "You are a helpful assistant capable of understanding a user\s needs through conversation to recommend suitable services. Based on the conversation history and the user'\''s last message, list services that can address the user'\''s needs. Respond only in Vietnamese or English, matching the language of the user'\''s input."
      },
      {
        "role": "user",
        "content": "Hi."
      }
    ],
    "stream": false
  }'
 
```
### Key Parameters Explained
- `model`: String identifier for the model you want to use
- `api_base`: The base URL endpoint for your API
- `api_key`: Your authentication API key
- `messages`: List of message objects with the following structure:
  - `role`: Can be "system", "user", or "assistant"
  - `content`: The actual message content
- `stream`: Boolean flag to enable streaming (set to `True` for streaming responses)
### Response Format
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
### Error Handling
The code includes try-catch block to handle potential errors during the API call and streaming process. Any errors will be returned as JSON-encoded error messages in the stream.
## Notes
- Make sure to handle the async nature of the function with appropriate async/await syntax
- The streaming response is formatted as Server-Sent Events (SSE)
- Each chunk is JSON-encoded and prefixed with "data: "
- The stream ends with a [DONE] marker

## List of Available Models

- DeepSeek-R1
- DeepSeek-R1-Distill-Llama-70B  
- DeepSeek-R1-Distill-Llama-8B  
- DeepSeek-R1-Distill-Qwen-1.5B  
- DeepSeek-R1-Distill-Qwen-32B  
- Llama-3.1-8B-Instruct  
- Llama-3.3-70B-Instruct  
- Llama-3-Swallow-70B-v0.1
- Llama-Guard-3-8B  
- Qwen2.5-7B-chat  
- Qwen2.5-Coder-32B-Instruct  
- SaoLa-Llama3.1-planner

> Note: You can access your subscribed models only.
