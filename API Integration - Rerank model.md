### cURL
```bash
curl --location 'https://mkp-api-stg.fptcloud.net/v1/rerank' \
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
