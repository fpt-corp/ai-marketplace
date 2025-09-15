> API Reference for Speech to text
# Bash script to process audio files
```
!/bin/bash

Install sox if not already installed

if ! command -v sox &> /dev/null; then

    echo "sox could not be found, installing..."
    
    sudo apt-get update
    
    sudo apt-get install -y sox
    
fi

Check if the input file is provided

if [ -z "$1" ]; then

    echo "Usage: $0 <input_audio_file>"
    
    exit 1
    
fi

Convert the audio file to WAV format

1 channel, 16kHz, 16-bit PCM

INPUT_FILE="$1"

OUTPUT_FILE="${INPUT_FILE%.*}_norm.wav"

sox "$INPUT_FILE" -c 1 -r 16000 -b 16 "$OUTPUT_FILE"
```
