> API Reference for Speech to text
# Python
```python
from openai import OpenAI

BASE_URL = "https://mkp-api.fptcloud.com"
API_KEY = "your-api-key"
MODEL_NAME = "your-model-name"
LANGUAGE = 'en'

client = OpenAI(api_key=API_KEY, 
                base_url=BASE_URL)

file = "speech.wav"
# file = "speech.mp3"
with open(file, "rb") as audio_file:
    data = audio_file.read()
    response = client.audio.transcriptions.create(
        model=MODEL_NAME,
        file=data,
        language=LANGUAGE,
        timeout=60,
        response_format="json"
    )

    print(response.text)
```
# cURL
```shell
curl --request POST \
  --url https://mkp-api.fptcloud.com/v1/audio/transcriptions \
  --header "Authorization: Bearer your-api-key" \
  --form "model=your-model-name" \
  --form "file=@your-file-path.mp3" \
  --form "language=en" \
  --form "response_format=json" \
  --max-time 60
```
# Preprocess audio files
> Bash script preprocessing utility for audio files

```bash
#!/bin/bash

# Install sox if not already installed
if ! command -v sox &> /dev/null; then
    echo "sox could not be found, installing..."
    sudo apt-get update
    sudo apt-get install -y sox
fi

# Check if the input file is provided
if [ -z "$1" ]; then
    echo "Usage: $0 <input_audio_file>"
    exit 1
fi

# Convert the audio file to WAV format
# 1 channel, 16kHz, 16-bit PCM
INPUT_FILE="$1"
OUTPUT_FILE="${INPUT_FILE%.*}_norm.wav"
sox "$INPUT_FILE" -c 1 -r 16000 -b 16 "$OUTPUT_FILE"
```
