# API Reference
This API reference describes the RESTful, streaming, and real-time APIs of AI Models on [AI Marketplace](https://marketplace.fptcloud.com/). REST APIs are usable via HTTP in any environment that supports HTTP requests. 
## Authentication 
The AI Marketplace API uses API keys for authentication. Create, manage, and learn more about API keys in [My account](https://marketplace.fptcloud.com/en/my-account).

> ⚠️ **Remember that your API key is a secret!**  
> Do not share it with others or expose it in any client-side code (browsers, apps).  
> API keys should be securely loaded from an environment variable or key management service on the server.

#### Authentication
  * All requests to the AI Marketplace API must include an `api-key` header with your API key. If you are using the Client SDKs, you will set the API when constructing a client, and then the SDK will send the header on your behalf with every request. If integrating directly with the API, you’ll need to send this header yourself.
  * Usage from these API requests counts as usage for the specified API Key and AI model.

## Large Language Model
[API Reference  for Large Language Model (LLM)](https://github.com/fpt-corp/ai-marketplace/blob/693f71389439e40e7a94ec5d21c02e17050a00f8/API%20Integration%20-%20Large%20Language%20Model.md)
## Vision Language Model
[API Reference for Vision Language Model]() 
## Multimodal Model
[API Reference for Multimodal Model]() 
## Embedding Model
[API Reference for Embedding Model]() 
