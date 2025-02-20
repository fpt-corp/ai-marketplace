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
async def stream_response():
    try:
        # Initialize the completion request
        response = await acompletion(
            model="your-model-name",              # The model you want to use
            api_base="your-api-endpoint",         # Base URL for API
            api_key="your-api-key",              # Your API key
            messages=[                            # List of message objects
                {
                    "role": "system",
                    "content": "Use english to answer all question."
                },
                {
                    "role": "user",
                    "content": "your-input-text"
                }
            ],
            stream=True                          # Enable streaming
        )
        # Process streaming response
        async for chunk in response:
            if chunk.choices[0].delta.content:
                yield f"data: {json.dumps({'content': chunk.choices[0].delta.content})}\n\n"
    except Exception as e:
        yield f"data: {json.dumps({'error': str(e)})}\n\n"
    yield "data: [DONE]\n\n"
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

# List of Available Models

- Llama3-Swallow-70B  
- DeepSeek-R1-Distill-Qwen-32B  
- DeepSeek-R1-Distill-Llama-70B  
- DeepSeek-R1-Distill-Llama-8B  
- SaoLa-Llama3.1-planner  
- Qwen2.5-7B-chat  
- DeepSeek-R1-Distill-Qwen-1.5B  
- DeepSeek-R1  
- Llama-Guard-3-8B  
- Llama-3.1-8B-Instruct  
- Qwen2.5-Coder-32B-Instruct  
- LLama-3.3-70B-Instruct  

> Note: You can access your subscribed models only.
