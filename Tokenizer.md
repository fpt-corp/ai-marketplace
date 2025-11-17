# Token Calculator — Usage Guide

_Công cụ tính token cho LLM và Embedding Models_

---

## 📚 Mục lục

1. **[Tổng quan](#tổng-quan)**
2. **[Cài đặt](#cài-đặt)**
3. **[LLM Token Calculator](#llm-token-calculator)**
   - [Cách đếm Input Tokens (HuggingFace Tokenizer)](#cách-đếm-input-tokens-theo-huggingface-tokenizer)
4. **[Embedding Token Calculator](#embedding-token-calculator)**
   - [Cách đếm Input Tokens (HuggingFace Tokenizer)](#cách-đếm-input-tokens-theo-huggingface-tokenizer-1)
5. **[Reference](#reference)**

---

## Tổng quan

Bộ công cụ gồm 2 calculator chuyên biệt:

### 1. LLM Token Calculator (`complete_token_calculator_llm.py`)

Dành cho **Chat Completion models** (Gemma, GPT, Claude, v.v.)

**So sánh 2 phương pháp:**
- ✅ **OpenAI Client** (baseline - ground truth)
- ✅ **LiteLLM** (wrapper library)

**Đặc điểm:**
- Đếm cả `prompt_tokens` và `completion_tokens`
- Input: Messages (chat format)
- Output: Response text + token usage

### 2. Embedding Token Calculator (`complete_token_calculator_embedding.py`)

Dành cho **Embedding models** (E5, text-embedding-ada-002, v.v.)

**So sánh 3 phương pháp:**
- ✅ **OpenAI Client** (baseline - ground truth)
- ✅ **LiteLLM** (wrapper library)
- ✅ **LangChain** (high-level framework)

**Đặc điểm:**
- Chỉ đếm `prompt_tokens` (embeddings không có completion)
- Input: Text hoặc list of texts
- Output: Embedding vectors + token usage

---

## Cài đặt

### Yêu cầu

```bash
Python >= 3.8
```

### Dependencies

```bash
# Core libraries
pip install litellm openai

# Optional: LangChain (cho embedding calculator)
pip install langchain-openai

# Optional: Custom tokenizer support
pip install transformers torch
```

---

## LLM Token Calculator

### Cách chạy

```bash
python complete_token_calculator_llm.py
```

### Sử dụng trong code

```python
from complete_token_calculator_llm import TokenCalculator

# Khởi tạo
calc = TokenCalculator(
    api_base="https://mkp-api.fptcloud.com",
    api_key="sk-your-api-key",
    model="openai/gemma-3-27b-it"
)

# Messages input
messages = [
    {"role": "user", "content": "Hello, how are you?"}
]

# Method 1: OpenAI Client (baseline)
openai_result = calc.calculate_tokens_openai(messages)
print(f"OpenAI: {openai_result.total_tokens} tokens")

# Method 2: LiteLLM
litellm_result = calc.calculate_tokens(messages)
print(f"LiteLLM: {litellm_result.total_tokens} tokens")
```

### Output mẫu

```
================================================================================
TOKEN CALCULATOR - LiteLLM with Custom Tokenizer
================================================================================

Response: 'Hello! How can I help you today?'

┌─────────────────────────┬───────────────┬─────────────────┬─────────────────┐
│ Method                  │    Prompt     │   Completion    │      Total      │
├─────────────────────────┼───────────────┼─────────────────┼─────────────────┤
│ MKP API (Production)    │      18       │       10        │       28        │
├─────────────────────────┼───────────────┼─────────────────┼─────────────────┤
│ LiteLLM                 │      18       │       10        │       28        │ ✓
└─────────────────────────┴───────────────┴─────────────────┴─────────────────┘
```

### Cơ chế hoạt động

#### Method 1: OpenAI Client (Baseline)

```
User → messages → OpenAI Client → API Call → Response
                                                 ↓
                                            usage.prompt_tokens
                                            usage.completion_tokens
                                            usage.total_tokens
```

```python
def calculate_tokens_openai(self, messages, **kwargs):
    # Gọi API qua OpenAI client
    client = OpenAI(api_key=self.api_key, base_url=self.api_base)
    response = client.chat.completions.create(
        model=self.model,
        messages=messages,
        **kwargs
    )

    # Lấy token usage từ response
    usage = response.usage
    return UsageResult(
        prompt_tokens=usage.prompt_tokens,
        completion_tokens=usage.completion_tokens,
        total_tokens=usage.total_tokens,
        ...
    )
```

#### Method 2: LiteLLM

```
User → messages → LiteLLM → API Call → Response
                                          ↓
                                    usage.prompt_tokens
                                    usage.completion_tokens
                                    usage.total_tokens
```

```python
def calculate_tokens(self, messages, **kwargs):
    # Gọi API qua LiteLLM
    response = litellm.completion(
        model=self.model,
        messages=messages,
        api_base=self.api_base,
        api_key=self.api_key,
        **kwargs
    )

    # Lấy token usage từ response
    usage = response.usage
    return UsageResult(
        prompt_tokens=usage.prompt_tokens,
        completion_tokens=usage.completion_tokens,
        total_tokens=usage.total_tokens,
        ...
    )
```

**Kết luận**: Cả 2 method đều gọi API thực và đều cho kết quả chính xác như nhau. LiteLLM là wrapper giúp code đơn giản hơn khi làm việc với nhiều provider.

### Cách đếm Input Tokens theo HuggingFace Tokenizer

Các API methods ở trên trả về số tokens từ server. Để hiểu rõ cách server đếm tokens, ta cần biết tokenizer mà model sử dụng.

#### Model: google/gemma-3-27b-it

**Tokenizer**: Gemma sử dụng **Gemma tokenizer** (dựa trên SentencePiece)

```python
from transformers import AutoTokenizer

# Load tokenizer từ HuggingFace
tokenizer = AutoTokenizer.from_pretrained("google/gemma-2-27b-it")

# Đếm tokens cho input
messages = [{"role": "user", "content": "Hello, how are you?"}]

# Apply chat template
formatted_prompt = tokenizer.apply_chat_template(
    messages,
    tokenize=False,
    add_generation_prompt=True
)

# Tokenize và đếm
tokens = tokenizer.encode(formatted_prompt)
num_tokens = len(tokens)

print(f"Input text: {formatted_prompt}")
print(f"Number of tokens: {num_tokens}")
print(f"Token IDs: {tokens}")
```

**Cơ chế đếm:**
1. Chat messages được format theo template của model (beg_of_turn, end_of_turn tokens)
2. Text được tokenize thành subword pieces
3. Special tokens được thêm vào: `<bos>`, `<eos>`, `<start_of_turn>`, `<end_of_turn>`

**Ví dụ với "Hello, how are you?":**
```
<bos><start_of_turn>user
Hello, how are you?<end_of_turn>
<start_of_turn>model

Tokens: [2, 106, 1645, 108, 4521, 235269, 1368, 708, 692, 235336, 107, 108, 106, 2516, 108]
Count: 15 tokens (bao gồm special tokens)
```

**Lưu ý:**
- API có thể trả về số tokens khác một chút do cách xử lý internal
- Special tokens và chat template format ảnh hưởng đến token count
- Gemma-2-27b-it và Gemma-3-27b-it có thể dùng cùng tokenizer

---

## Embedding Token Calculator

### Cách chạy

```bash
python complete_token_calculator_embedding.py
```

### Sử dụng trong code

```python
from complete_token_calculator_embedding import EmbeddingTokenCalculator

# Khởi tạo
calc = EmbeddingTokenCalculator(
    api_base="https://mkp-api.fptcloud.com",
    api_key="sk-your-api-key",
    model="multilingual-e5-large"
)

# Text input
text = "Hello, how are you?"

# Method 1: OpenAI Client (baseline)
openai_result = calc.calculate_litellm_tokens_openai(text)
print(f"OpenAI: {openai_result.total_tokens} tokens")

# Method 2: LiteLLM
litellm_result = calc.calculate_litellm_tokens(text)
print(f"LiteLLM: {litellm_result.total_tokens} tokens")

# Method 3: LangChain (optional)
langchain_result = calc.calculate_litellm_tokens_langchain(text)
print(f"LangChain: {langchain_result.total_tokens} tokens")
```

### Output mẫu

```
==================================================================================
EMBEDDING TOKEN CALCULATOR - intfloat/multilingual-e5-large
Comparison: OpenAI Client vs LiteLLM vs LangChain
==================================================================================

Input: 'Hello, how are you?'

Testing 3 methods...

1️⃣  OpenAI Client (Baseline - Ground Truth)
   Result: 19 tokens
   Response time: 371.82ms

2️⃣  LiteLLM API
   Result: 19 tokens
   Response time: 226.84ms

3️⃣  LangChain OpenAIEmbeddings
   Result: 19 tokens
   Response time: 213.59ms

==================================================================================
COMPARISON TABLE
==================================================================================

┌──────────────────────────────┬───────────────┬───────────────┬────────────────────┐
│ Method                       │ Prompt Tokens │  Total Tokens │     Difference     │
├──────────────────────────────┼───────────────┼───────────────┼────────────────────┤
│ OpenAI Client (Baseline)     │       19      │       19      │         -          │
├──────────────────────────────┼───────────────┼───────────────┼────────────────────┤
│ MKP API (Production)         │       19      │       19      │         ✓          │
├──────────────────────────────┼───────────────┼───────────────┼────────────────────┤
│ LangChain OpenAIEmbeddings   │       19      │       19      │         ✓          │
└──────────────────────────────┴───────────────┴───────────────┴────────────────────┘
```

### Cơ chế hoạt động

#### Method 1: OpenAI Client (Baseline)

```
User → text → OpenAI Client → embeddings.create() → Response
                                                        ↓
                                                  usage.prompt_tokens
                                                  usage.total_tokens
```

```python
def calculate_litellm_tokens_openai(self, input_texts, **kwargs):
    # Gọi API qua OpenAI client
    client = OpenAI(api_key=self.api_key, base_url=self.api_base)
    response = client.embeddings.create(
        model=self.model,
        input=input_texts,
        **kwargs
    )

    # Lấy token usage từ response
    usage = response.usage
    return EmbeddingUsageResult(
        prompt_tokens=usage.prompt_tokens,
        total_tokens=usage.total_tokens,
        ...
    )
```

#### Method 2: LiteLLM

```
User → text → LiteLLM → litellm.embedding() → Response
                                                  ↓
                                            usage.prompt_tokens
                                            usage.total_tokens
```

```python
def calculate_litellm_tokens(self, input_texts, **kwargs):
    # Gọi API qua LiteLLM
    response = litellm.embedding(
        model=self.model,
        input=input_texts,
        api_base=self.api_base,
        api_key=self.api_key,
        **kwargs
    )

    # Lấy token usage từ response
    usage = response.usage
    return EmbeddingUsageResult(
        prompt_tokens=usage.prompt_tokens,
        total_tokens=usage.total_tokens,
        ...
    )
```

#### Method 3: LangChain

```
User → text → LangChain → OpenAIEmbeddings → underlying client → Response
                                                                     ↓
                                                               usage.prompt_tokens
                                                               usage.total_tokens
```

```python
def calculate_litellm_tokens_langchain(self, input_texts, **kwargs):
    # Tạo LangChain wrapper
    embeddings = OpenAIEmbeddings(
        model=self.model,
        openai_api_base=self.api_base,
        openai_api_key=self.api_key
    )

    # Truy cập underlying client của LangChain
    embeddings_client = embeddings.client

    # Gọi API trực tiếp để lấy usage info
    response = embeddings_client.create(
        model=self.model,
        input=texts_list,
        **kwargs
    )

    # Lấy token usage từ response
    usage = response.usage
    return EmbeddingUsageResult(
        prompt_tokens=usage.prompt_tokens,
        total_tokens=usage.total_tokens,
        ...
    )
```

**Kết luận**: Tất cả 3 methods đều gọi API thực và cho kết quả giống nhau. Chọn method phù hợp với stack công nghệ bạn đang dùng:
- **OpenAI Client**: Dùng khi chỉ cần OpenAI API
- **LiteLLM**: Dùng khi cần support nhiều providers
- **LangChain**: Dùng khi đã có hệ thống LangChain

### Cách đếm Input Tokens theo HuggingFace Tokenizer

Các API methods ở trên trả về số tokens từ AI Serving Engine. Để hiểu rõ cách AI Engine đếm tokens, ta cần biết tokenizer mà model sử dụng.
- **Gemma Models**: https://huggingface.co/google/gemma-2-27b-it
- **E5 Embedding Models**: https://huggingface.co/intfloat/multilingual-e5-large

## Reference

### API & Libraries
- **LiteLLM Docs**: https://docs.litellm.ai/
- **OpenAI API**: https://platform.openai.com/docs
- **LangChain Docs**: https://python.langchain.com/docs/

### Tokenizers
- **HuggingFace Transformers**: https://huggingface.co/docs/transformers/
- **Tokenizers Guide**: https://huggingface.co/docs/transformers/tokenizer_summary

### Tokenizer Implementation
- **SentencePiece**: https://github.com/google/sentencepiece
- **BPE Algorithm**: https://en.wikipedia.org/wiki/Byte_pair_encoding
- **XLM-RoBERTa**: https://huggingface.co/xlm-roberta-large
