# API Reference
This API reference describes the RESTful, streaming, and real-time APIs of AI Models on [AI Marketplace](https://marketplace.fptcloud.com/). REST APIs are usable via HTTP in any environment that supports HTTP requests. 

## Authentication 
The AI Marketplace API uses API keys for authentication. Create, manage, and learn more about API keys in [My account/ My API Keys](https://marketplace.fptcloud.com/en/my-account#my-api-key).

> ⚠️ **Remember that your API key is a secret!**  
> Do not share it with others or expose it in any client-side code (browsers, apps).  
> API keys should be securely loaded from an environment variable or key management service on the server.

#### Authentication
  * All requests to the AI Marketplace API must include an `api-key` header with your API key. If you are using the Client SDKs, you will set the API when constructing a client, and then the SDK will send the header on your behalf with every request. If integrating directly with the API, you’ll need to send this header yourself.
  * Usage from these API requests counts as usage for the specified API Key and AI model.

## Large Language Model
[API Reference  for Large Language Model (LLM)](API Integration - Large Language Model.md)

## Vision Language Model
[API Reference for Vision Language Model (VLM)](API Integration - Vision Language Model.md) 

## Multimodal Model
API Reference for Multimodal Model
  - [Image](API Integration - Multimodal Model - Image.md)
  - [Text](API Integration - Multimodal Model - Text.md)

## Embedding Model
[API Reference for Embedding Model](API Integration - Embedding Model.md) 

## List of Available Models

- DeepSeek-R1
- DeepSeek-R1-Distill-Llama-70B  
- DeepSeek-R1-Distill-Llama-8B  
- DeepSeek-R1-Distill-Qwen-1.5B  
- DeepSeek-R1-Distill-Qwen-32B  
- Llama-3.1-8B-Instruct  
- Llama-3.3-70B-Instruct  
- Llama-3.3-Swallow-70B-Instruct-v0.4
- Llama-Guard-3-8B
- QwQ-32B
- Qwen2.5-7B-instruct  
- Qwen2.5-Coder-32B-Instruct
- SaoLa3.1-medium
- SaoLa-Llama3.1-planner
- Vietnamese_Embedding
- FPT.AI-e5-large
- FPT.AI-gte-base
- Qwen2.5-VL-7B-Instruct
- Llama-4-Scout-17B-16E
