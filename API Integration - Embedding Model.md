> API Reference for Embedding Model
# Using Langchain
```
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
