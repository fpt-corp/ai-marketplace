> API Reference for Vision Language Model (VLM)
# Python
```python
#!/usr/bin/env python3

# Import required libraries
import base64
import requests
import io
from io import BytesIO
from PIL import Image
from openai import OpenAI

# === Configuration ===
API_KEY = ""
BASE_URL = "https://mkp-api.fptcloud.com"                        # Base URL for API
MODEL="{model-name}"                                             # Model name
image_file = "Vinh-Ha-Long-1.jpg"                                # Image file

client = OpenAI(
    api_key=API_KEY,
    base_url=BASE_URL
)

def encode_image(image_path: str, resize=False, size=(1280, 1280)) -> str:
    """
    Encodes an image file to a base64 string.
    Args:
        image_path (str): The path to the image file.
    """
    if resize:
        with Image.open(image_path) as img:
            img = img.resize(size)
            # conver img to base64
            buffered = io.BytesIO()
            if image_path.endswith(".jpg") or image_path.endswith(".jpeg"):
                img.save(buffered, format="JPEG")
                format = "jpeg"
            elif image_path.endswith(".png"):
                img.save(buffered, format="PNG")
                format = "png"
            else:
                raise ValueError("Unsupported image format")
            encoded_string = base64.b64encode(buffered.getvalue()).decode("utf-8")
    else:
        with open(image_path, "rb") as image_file:
            encoded_string = base64.b64encode(image_file.read()).decode("utf-8")

        if image_path.endswith(".jpg") or image_path.endswith(".jpeg"):
            format = "jpeg"
        elif image_path.endswith(".png"):
            format = "png"
        else:
            raise ValueError("Unsupported image format")
    
    return encoded_string, format

def encode_image_content_from_url(image_url: str, resize=False, size=(1280, 1280)) -> str:
    """
    Encode an image from a URL to a base64 string.
    Args:
        image_url (str): The URL of the image.
    Returns:
        str: The base64 encoded string of the image.
    """
    if resize:
        with requests.get(image_url, stream=True) as response:
            response.raise_for_status()
            img = Image.open(BytesIO(response.content))
            img = img.resize(size)
            buffered = BytesIO()
            img.save(buffered, format="PNG")
            encoded_string = base64.b64encode(buffered.getvalue()).decode("utf-8")
        return encoded_string
    else:
        with requests.get(image_url) as response:
            response.raise_for_status()
            encoded_string = base64.b64encode(response.content).decode("utf-8")
    return encoded_string

def run_single_image_from_file(img_path: str):
    """
    Run the VLM model on a single image from a file.
    Args:
        img_path (str): The path to the image file.
    """
    encoded_img, format  = encode_image(img_path, resize=True, size=(1280, 1280))
    # print(encoded_img)

    # Initialize the completion request                           
    chat_completion = client.chat.completions.create(
        messages=[                                            # List of message objects. Please update the System prompt to have the model respond appropriately
            {
                "role": "system",
                "content": "You are an AI assistant that can describe images. Provide detailed descriptions. Use bullet points if necessary. Provide your answer in Vietnamese. Do not include any other text or instructions. Only provide the description of the image.",
            },
            {
                "role": "user",
                "content": [
                    {
                        "type": "text",
                        "text": "Bạn có thể mô tả hình ảnh này không?",
                    },
                    {
                        "type": "image_url",
                        "image_url": f"data:image/{format};base64,{encoded_img}"
                    }
                ],
            },
        ],
        model=MODEL,
        temperature=0.0,
        stream=True                                             # Enable streaming to receive response chunks
    )

    # Process streaming response
    for chunk in chat_completion:
        if chunk is not None:
            print(chunk.choices[0].delta.content, end='', flush=True)
    print("")

def run_single_image_from_url(url: str):
    """
    Run the VLM model on a single image from a URL.
    Args:
        img_path (str): The path to the image file.
    """
    # Initialize the completion request  
    chat_completion = client.chat.completions.create(
        messages=[                                              # List of message objects. Please update the System prompt to have the model respond appropriately
            {
                "role": "system",
                "content": "You are an AI assistant that can describe images. Provide detailed descriptions. Use bullet points if necessary. Provide your answer in Vietnamese. Do not include any other text or instructions. Only provide the description of the image.",
            },
            {
                "role": "user",
                "content": [
                    {
                        "type": "text",
                        "text": "Mô tả bức ảnh ?",
                    },
                    {
                        "type": "image_url",
                        "image_url": {
                            "url": f"{url}"
                        }
                    }
                ],
            },
        ],
        model=MODEL,
        temperature=0.0,
        stream=True                                               # Enable streaming to receive response chunks
    )

    # Process streaming response
    for chunk in chat_completion:
        if chunk is not None:
            print(chunk.choices[0].delta.content, end='', flush=True)
    print("")

# Execute the function to process a local image file
run_single_image_from_file(image_file)
# Uncomment the line below to process an image from a URL instead
# run_single_image_from_url(image_link)
```
