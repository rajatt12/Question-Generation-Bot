# 🤖 Question Generation POC

> A Proof of Concept for automatically generating meaningful questions from any text input using AI/NLP techniques.

---

## 📌 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Methods Implemented](#methods-implemented)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [Usage](#usage)
- [API Reference](#api-reference)
- [Configuration](#configuration)
- [Examples](#examples)
- [Roadmap](#roadmap)
- [Contributing](#contributing)
- [License](#license)

---

## Overview

This POC demonstrates the feasibility of **automated question generation** from raw text, documents, or structured data. It explores multiple generation strategies — from simple LLM prompting to RAG-based pipelines — to validate the approach before full-scale development.

**Use cases include:**
- E-learning & quiz generation
- HR interview question generation from job descriptions
- Knowledge base FAQ generation
- Document comprehension assessment
- Chatbot training data creation

---

## Features

- ✅ Paste or upload any text/document
- ✅ Generate questions using multiple strategies
- ✅ Support for different question types: MCQ, open-ended, factoid
- ✅ Difficulty-level tagging (Easy / Medium / Hard)
- ✅ Deduplication and ranking of output questions
- ✅ REST API endpoint for integration
- ✅ Simple web UI for testing

---

## Methods Implemented

| Method | Description | Best For |
|---|---|---|
| **LLM Prompt-Based** | Sends text to Claude/GPT with a crafted prompt | General-purpose, fast POC |
| **Fine-tuned T5/BART** | Uses Hugging Face seq2seq models (e.g., `valhalla/t5-base-qg-hl`) | Offline, factoid questions |
| **RAG-Based** | Chunks large docs → embeds → generates per chunk | PDFs, large knowledge bases |
| **NER + Templates** | Extracts named entities and fills question templates | Rule-based, interpretable |
| **Answer-Aware** | Identifies answer spans first, then generates questions | MCQ, exam prep |

---

## Tech Stack

- **Language:** Python 3.10+
- **LLM API:** Anthropic Claude / OpenAI GPT
- **NLP:** Hugging Face Transformers, spaCy
- **Embeddings / Vector Store:** FAISS / ChromaDB
- **Backend:** FastAPI
- **Frontend:** React.js (or plain HTML/JS for POC)
- **Others:** LangChain, sentence-transformers

---

## Project Structure

```
question-generation-poc/
├── app/
│   ├── api/
│   │   └── routes.py          # FastAPI route definitions
│   ├── core/
│   │   ├── llm_generator.py   # Method 1: LLM Prompt-Based
│   │   ├── t5_generator.py    # Method 2: Fine-tuned T5/BART
│   │   ├── rag_generator.py   # Method 3: RAG-Based pipeline
│   │   ├── ner_generator.py   # Method 4: NER + Templates
│   │   └── answer_aware.py    # Method 5: Answer-Aware Generation
│   ├── utils/
│   │   ├── preprocessor.py    # Text cleaning & chunking
│   │   └── postprocessor.py   # Deduplication & ranking
│   └── main.py                # App entry point
├── frontend/
│   └── index.html             # Simple UI for testing
├── tests/
│   └── test_generators.py
├── .env.example
├── requirements.txt
├── Dockerfile
└── README.md
```

---

## Getting Started

### Prerequisites

- Python 3.10+
- Node.js 18+ (for frontend)
- An Anthropic or OpenAI API key

### Installation

```bash
# 1. Clone the repository
git clone https://github.com/your-username/question-generation-poc.git
cd question-generation-poc

# 2. Create a virtual environment
python -m venv venv
source venv/bin/activate      # On Windows: venv\Scripts\activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. Set up environment variables
cp .env.example .env
# Add your API keys in .env

# 5. Run the application
uvicorn app.main:app --reload
```

### Environment Variables

Create a `.env` file in the root directory:

```env
ANTHROPIC_API_KEY=your_anthropic_api_key_here
OPENAI_API_KEY=your_openai_api_key_here       # Optional
HF_MODEL_NAME=valhalla/t5-base-qg-hl          # For Method 2
EMBEDDING_MODEL=all-MiniLM-L6-v2
MAX_QUESTIONS=10
DEFAULT_METHOD=llm_prompt
```

---

## Usage

### Via Web UI

Open `http://localhost:8000` in your browser, paste your text, select a method and question type, and click **Generate**.

### Via API

```bash
curl -X POST http://localhost:8000/api/generate \
  -H "Content-Type: application/json" \
  -d '{
    "text": "The mitochondria is the powerhouse of the cell...",
    "method": "llm_prompt",
    "question_type": "open_ended",
    "num_questions": 5
  }'
```

### Via Python SDK

```python
from app.core.llm_generator import LLMQuestionGenerator

generator = LLMQuestionGenerator()
questions = generator.generate(
    text="Your input text here...",
    num_questions=5,
    question_type="mcq"
)

for q in questions:
    print(q)
```

---

## API Reference

### `POST /api/generate`

Generates questions from input text.

**Request Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `text` | string | ✅ | Input text to generate questions from |
| `method` | string | ❌ | One of: `llm_prompt`, `t5`, `rag`, `ner`, `answer_aware` (default: `llm_prompt`) |
| `question_type` | string | ❌ | One of: `open_ended`, `mcq`, `factoid` (default: `open_ended`) |
| `num_questions` | integer | ❌ | Number of questions to generate (default: 5) |
| `difficulty` | string | ❌ | One of: `easy`, `medium`, `hard`, `mixed` (default: `mixed`) |

**Response:**

```json
{
  "questions": [
    {
      "id": 1,
      "question": "What is the primary function of mitochondria?",
      "type": "open_ended",
      "difficulty": "easy",
      "source_chunk": "The mitochondria is the powerhouse of the cell..."
    }
  ],
  "method_used": "llm_prompt",
  "total_generated": 5,
  "processing_time_ms": 1240
}
```

---

## Examples

### Input Text

```
Photosynthesis is the process by which green plants and some other organisms use
sunlight to synthesize nutrients from carbon dioxide and water. It generates oxygen
as a byproduct and is essential for life on Earth.
```

### Generated Questions

```
1. What is photosynthesis and how does it work?
2. What are the two main inputs required for photosynthesis?
3. What byproduct is produced during photosynthesis?
4. Why is photosynthesis considered essential for life on Earth?
5. Which organisms are capable of performing photosynthesis?
```

---

## Architecture

```
Input Text / Document
        │
        ▼
  Preprocessor
  (clean, chunk, detect language)
        │
        ▼
  Question Generation Engine
  ┌─────────────────────────────┐
  │  LLM │ T5 │ RAG │ NER │ AA │
  └─────────────────────────────┘
        │
        ▼
  Postprocessor
  (deduplicate, rank, tag difficulty)
        │
        ▼
  Output: Question List
  (with type, difficulty, source)
```

---

## Roadmap

- [ ] Multi-language support
- [ ] PDF and DOCX file upload
- [ ] Answer generation alongside questions
- [ ] MCQ distractor generation
- [ ] Export to CSV / JSON / Anki format
- [ ] Batch processing via file upload
- [ ] Fine-tuned model on domain-specific data
- [ ] Streamlit UI as an alternative frontend

---

## Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository
2. Create a new branch (`git checkout -b feature/your-feature-name`)
3. Make your changes and write tests
4. Commit your changes (`git commit -m 'Add: your feature description'`)
5. Push to the branch (`git push origin feature/your-feature-name`)
6. Open a Pull Request

Please make sure your code passes all existing tests before submitting.

---

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

---

## Acknowledgements

- [Anthropic Claude API](https://www.anthropic.com)
- [Hugging Face Transformers](https://huggingface.co/transformers/)
- [LangChain](https://www.langchain.com/)
- [FastAPI](https://fastapi.tiangolo.com/)
- [spaCy](https://spacy.io/)

---

> Built with ❤️ as a Proof of Concept. Not production-ready — use as a starting point for further development.
