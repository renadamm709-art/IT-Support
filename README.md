# IT Support Workflow

## Overview

This workflow builds an IT Support assistant in n8n. It downloads the IT Support PDF from Google Drive, extracts its content, stores it in an in-memory vector store, and uses an AI Agent to answer user questions based on the retrieved guide.

## Workflow Components

### 1. Download file
- Node type: `n8n-nodes-base.googleDrive`
- Operation: `download`
- Source file: `IT Support.pdf`
- Google Drive file ID: `1jYd6iDzeBELIbYRGaS9OOcW6mvVaXxht`

### 2. Extract from File
- Node type: `n8n-nodes-base.extractFromFile`
- Operation: `pdf`
- Purpose: Extracts text/content from the downloaded PDF.

### 3. Simple Vector Store
- Node type: `@n8n/n8n-nodes-langchain.vectorStoreInMemory`
- Mode: `insert`
- Memory key: `vector_store_key`
- Purpose: Stores the extracted IT Support Guide content for semantic retrieval.

### 4. Default Data Loader
- Node type: `@n8n/n8n-nodes-langchain.documentDefaultDataLoader`
- Data type: `binary`
- Purpose: Converts the extracted file data into documents for the vector store.

### 5. Embeddings OpenAI
- Node type: `@n8n/n8n-nodes-langchain.embeddingsOpenAi`
- Purpose: Generates embeddings for documents inserted into the vector store.

### 6. When chat message received
- Node type: `@n8n/n8n-nodes-langchain.chatTrigger`
- File uploads: enabled
- Purpose: Receives user chat messages and starts the AI support flow.

### 7. AI Agent
- Node type: `@n8n/n8n-nodes-langchain.agent`
- Purpose: Acts as the IT Support assistant.
- System instruction: Always use the IT Support Guide tool to search for relevant information before answering the user's question, and base the answer on the retrieved information.

### 8. OpenAI Chat Model
- Node type: `@n8n/n8n-nodes-langchain.lmChatOpenAi`
- Model: `gpt-5-mini`
- Purpose: Provides the language model used by the AI Agent.

### 9. Simple Memory
- Node type: `@n8n/n8n-nodes-langchain.memoryBufferWindow`
- Context window length: `10`
- Purpose: Maintains recent conversation context for the AI Agent.

### 10. Simple Vector Store1
- Node type: `@n8n/n8n-nodes-langchain.vectorStoreInMemory`
- Mode: `retrieve-as-tool`
- Memory key: `vector_store_key`
- Top K: `200`
- Purpose: Exposes the IT Support Guide as a retrieval tool for the AI Agent.
- Tool description: Search the IT Support Guide for relevant information and always use this tool before responding.

### 11. Embeddings OpenAI1
- Node type: `@n8n/n8n-nodes-langchain.embeddingsOpenAi`
- Purpose: Generates embeddings used by the retrieval vector store.

## Connections

```text
Download file
    ↓
Extract from File
    ↓
Simple Vector Store
    ↑
Default Data Loader
    ↑
Embeddings OpenAI

When chat message received
    ↓
AI Agent
    ├── OpenAI Chat Model
    ├── Simple Memory
    └── Simple Vector Store1
             ↑
       Embeddings OpenAI1
```

## Data Flow

1. The workflow downloads `IT Support.pdf` from Google Drive.
2. The PDF is processed by `Extract from File`.
3. The extracted content is loaded into the in-memory vector store.
4. OpenAI embeddings are used to represent the stored documents.
5. A user sends a message through the chat trigger.
6. The AI Agent receives the message and uses the IT Support Guide retrieval tool.
7. The vector store returns relevant information from the guide.
8. The AI Agent uses `gpt-5-mini` and the retrieved information to formulate the response.
9. `Simple Memory` keeps the last 10 context items available to the agent.

## Configuration Notes

- Workflow active status: `false`.
- Execution order: `v1`.
- Binary mode: `separate`.
- Available in MCP: `false`.
- The workflow uses OpenAI credentials for both the chat model and embeddings.
- The retrieval tool is explicitly configured to be used before answering IT Support questions.
