> API Reference for ReRank model
### cURL
```bash
curl --location 'https://mkp-api.fptcloud.com/v1/rerank' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer your-api-key' \
--data '
{
"model": "bge-reranker-v2-m3",
"query": "What is the capital of the United States?",
"documents": [
"Carson City is the capital city of the American state of Nevada.",
"The Commonwealth of the Northern Mariana Islands is a group of islands in the Pacific Ocean. Its capital is Saipan.",
"Washington, D.C. is the capital of the United States.",
"Capital punishment has existed in the United States since before it was a country."
],
"top_n": 2
}'
```
### Python
```python
from openai import OpenAI

client = OpenAI(
    base_url="https://mkp-api.fptcloud.com",
    api_key="your_api_key"
)

def rerank_documents(query, documents, model="bge-reranker-v2-m3", top_n=2):
    """Rerank documents using custom request."""
    
    # Access the underlying httpx client
    response = client._client.post(
        f"{client.base_url}/v1/rerank",
        json={
            "model": model,
            "query": query,
            "documents": documents,
            "top_n": top_n
        },
        headers={
            "Authorization": f"Bearer {client.api_key}",
            "Content-Type": "application/json"
        }
    )
    
    return response.json()

# Example usage
if __name__ == "__main__":
    query = "What is the capital of the United States?"
    
    documents = [
        "Carson City is the capital city of the American state of Nevada.",
        "The Commonwealth of the Northern Mariana Islands is a group of islands in the Pacific Ocean. Its capital is Saipan.",
        "Washington, D.C. is the capital of the United States.",
        "Capital punishment has existed in the United States since before it was a country."
    ]
    
    result = rerank_documents(query, documents, top_n=2)
    print("Rerank Results:")
    print(result)
```
### Nodejs
```js
const axios = require('axios');

class RerankClient {
    constructor(baseUrl, apiKey) {
        this.baseUrl = baseUrl;
        this.apiKey = apiKey;
    }

    async rerankDocuments(query, documents, model = "bge-reranker-v2-m3", topN = 2) {
        try {
            const response = await axios.post(
                `${this.baseUrl}/v1/rerank`,
                {
                    model: model,
                    query: query,
                    documents: documents,
                    top_n: topN
                },
                {
                    headers: {
                        "Authorization": `Bearer ${this.apiKey}`,
                        "Content-Type": "application/json"
                    }
                }
            );
            
            return response.data;
        } catch (error) {
            console.error("Error reranking documents:", error.message);
            throw error;
        }
    }
}

// Example usage
async function main() {
    const client = new RerankClient(
        "https://mkp-api.fptcloud.com",
        "your_api_key"
    );
    
    const query = "What is the capital of the United States?";
    
    const documents = [
        "Carson City is the capital city of the American state of Nevada.",
        "The Commonwealth of the Northern Mariana Islands is a group of islands in the Pacific Ocean. Its capital is Saipan.",
        "Washington, D.C. is the capital of the United States.",
        "Capital punishment has existed in the United States since before it was a country."
    ];
    
    try {
        const result = await client.rerankDocuments(query, documents, "bge-reranker-v2-m3", 2);
        console.log("Rerank Results:");
        console.log(JSON.stringify(result, null, 2));
    } catch (error) {
        console.error("Failed to rerank documents:", error);
    }
}

// Run the example
main();
```
