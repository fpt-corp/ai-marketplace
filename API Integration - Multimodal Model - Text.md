> API Reference for Multimodal Model - Text
# Python
```python
#!/bin/python3

# Import required libraries
import requests
import json

# === Configuration ===
API_KEY = ""
URL = "https://mkp-api.fptcloud.com/v1"                          # Base URL for API
MODEL="{model-name}"                                          # Model name
PROMPT = "Code fibonaci in C++?"                              # The user prompt or question to send to the model

headers = {
    "Authorization": f"Bearer {API_KEY}",
    "Content-Type": "application/json"
}
data = {
    "model": MODEL,
    "messages": [{"role": "user", "content": PROMPT}],
    "temperature": 0.3
}
response = requests.post(f"{URL}/chat/completions", headers=headers, data=json.dumps(data), stream=True)

if response.status_code == 200:
    print("Response received successfully.")
else:
    print(f"Failed to receive response. Status code: {response.status_code}")
for chunk in response.iter_content(chunk_size=None):
    if chunk:
        json_data = json.loads(chunk.decode('utf-8'))
        print(json_data['choices'][0]['message']['content'], end='')
print("")
```
