> API Reference for Embedding Model

# Python
```python
#!/usr/bin/env python3
import requests
import json
import os

API_KEY = ""
BASE_URL = "https://mkp-api.fptcloud.com"
MODEL = "{model-name}" # Your specific embedding model name

# Your long input text (same potential length issue applies!)
TEXT = "Xin chào, tôi là một mô hình ngôn ngữ lớn. Tôi có thể giúp gì cho bạn hôm nay " * 390 # ~ 8192 tokens

def get_embedding_requests(text: str, model: str, api_key: str, base_url: str) -> list[float] | None:
    # Construct the full API endpoint URL (assuming standard OpenAI path)
    endpoint_url = f"{base_url.rstrip('/')}/embeddings"

    # Prepare the request headers
    headers = {
        "Authorization": f"Bearer {api_key}", # Standard Bearer token authentication
        "Content-Type": "application/json",   # Specify JSON payload
    }

    # Prepare the request payload (body) in JSON format
    payload = {
        "input": [text],
        "model": model
    }

    print(f"Sending POST request to: {endpoint_url}")
    print(f"Using model: {model}")
    try:
        # Make the POST request
        response = requests.post(endpoint_url, headers=headers, json=payload) # Use json=payload for auto-serialization
        response.raise_for_status()

        response_data = response.json()
        # formated response and print it
        print(json.dumps(response_data, indent=2))

    except Exception as e:
        # Catch any other unexpected errors
        print(f"An unexpected error occurred: {e}")
        return None

# --- Get the embedding using raw requests ---
embedding_vector = get_embedding_requests(TEXT, MODEL, API_KEY, BASE_URL)
if embedding_vector:
    print(f"\nEmbedding vector received (first 10 dimensions): {embedding_vector[:10]}...")
    print(f"Embedding vector dimension: {len(embedding_vector)}")
else:
    print("\nFailed to retrieve the embedding vector using raw requests.")
```

> # Using Langchain
```python
#!/usr/bin/env python3

# Import required libraries
import os
from langchain_openai import OpenAIEmbeddings 

# === Configuration ===
API_KEY = "" 
BASE_URL = "https://mkp-api.fptcloud.com" 
MODEL = "{model-name}"                         # Your specific embedding model

TEXT = "Xin chào, tôi là một mô hình ngôn ngữ lớn. Tôi có thể giúp gì cho bạn hôm nay " * 390 # ~ 8192 tokens

try:
    # Configure OpenAIEmbeddings to use your custom endpoint and model
    # Langchain uses openai_api_base and openai_api_key for compatibility
    embeddings_model = OpenAIEmbeddings(
        model=MODEL,
        openai_api_key=API_KEY,
        openai_api_base=BASE_URL,
        # You might need to add other parameters depending on the endpoint's requirements
        # e.g., chunk_size if handling very long texts automatically (though your text might exceed single input limits)
    )
    print(f"Initialized Langchain OpenAIEmbeddings for model '{MODEL}' at '{BASE_URL}'")

except Exception as e:
    print(f"Error initializing Langchain Embeddings: {e}")
    exit()

def get_embedding_langchain(text: str, embeddings_model: OpenAIEmbeddings) -> list[float] | None:
    """
    Retrieves the embedding for a given text using the specified Langchain embeddings model.

    Args:
        text (str): The input text to embed.
        embeddings_model (OpenAIEmbeddings): The initialized Langchain embeddings model instance.

    Returns:
        list[float] | None: The embedding vector, or None if an error occurs.

    Note: This function expects the API to handle the input text length.
          Very long texts might exceed model/API limits.
    """
    try:
        print(f"Requesting embedding for text (length: {len(text)} chars)...")
        # Langchain's embed_documents expects a list of texts and returns a list of embeddings
        embedding_list = embeddings_model.embed_documents([text])

        # Check if we got a result
        if embedding_list and len(embedding_list) > 0:
             print("Successfully retrieved embedding.")
             return embedding_list[0] # Return the first (and only) embedding
        else:
            print("Error: Received empty embedding list from the API.")
            return None

    except Exception as e:
        print(f"An error occurred while getting embedding: {e}")
        return None

embedding_vector = get_embedding_langchain(TEXT, embeddings_model)
if embedding_vector:
    print(f"Embedding vector received (first 10 dimensions): {embedding_vector[:10]}...")
    print(f"Embedding vector dimension: {len(embedding_vector)}")
else:
    print("Failed to retrieve the embedding vector.")
```
# OpenAI
```python
#!/bin/python3
from openai import OpenAI
from transformers import AutoTokenizer

API_KEY = ""
BASE_URL = "https://mkp-api.fptcloud.com"
MODEL="{model-name}"

TEXT="Xin chào, tôi là một mô hình ngôn ngữ lớn. Tôi có thể giúp gì cho bạn hôm nay " * 390 # ~ 8192 tokens

client = OpenAI(
    api_key=API_KEY,
    base_url=BASE_URL
)

def get_embedding(text: str, model: str) -> list:
    """
    Retrieves the embedding for a given text using the specified model.
    Args:
        text (str): The input text to embed.
        model (str): The model to use for embedding.
    Returns:
        list: The embedding vector.
    """
    response = client.embeddings.create(
        input=text,
        model=model
    )

    return response.data[0].embedding

# get the embedding for TEXT
embedding_vector = get_embedding(TEXT, MODEL)
```
# Nodejs
```python
const OpenAI = require('openai');

const API_KEY = "";
const BASE_URL = "https://mkp-api.fptcloud.com";
const MODEL = "Vietnamese_Embedding"; 
const TEXT = "Xin chào, tôi là một mô hình ngôn ngữ lớn. Tôi có thể giúp gì cho bạn hôm nay " 

const openai = new OpenAI({ apiKey: API_KEY, baseURL: BASE_URL});

async function getEmbeddingWithOpenAI(text, model) {
    try {
        const response = await openai.embeddings.create({
            model: model,
            input: text, // OpenAI typically accepts an array of strings as input
        });

        if (response.data && response.data.length > 0) {
            return response.data[0].embedding;
        } else {
            console.error("Failed to retrieve embedding from OpenAI response.");
            return null;
        }
    } catch (error) {
        console.error("Error retrieving embedding from OpenAI:", error);
        return null;
    }
}

async function main() {
    const embeddingVector = await getEmbeddingWithOpenAI(TEXT, MODEL);

    if (embeddingVector) {
        console.log("\nEmbedding vector received (first 10 dimensions):", embeddingVector.slice(0, 10), "...");
        console.log("Embedding vector dimension:", embeddingVector.length);
    } else {
        console.log("\nFailed to retrieve the embedding vector using the OpenAI library.");
    }
}

main();
