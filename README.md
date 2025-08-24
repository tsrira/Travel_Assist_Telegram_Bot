# Panicker's Travel Assist Chatbot

[![n8n](https://img.shields.io/badge/Built%20with-n8n-FF5533.svg)](https://n8n.io/)
[![Telegram](https://img.shields.io/badge/Platform-Telegram-2CA5E0.svg)](https://telegram.org/)
[![OpenAI](https://img.shields.io/badge/Powered%20by-OpenAI-412991.svg)](https://openai.com/)
[![Pinecone](https://img.shields.io/badge/Vector%20DB-Pinecone-blue.svg)](https://www.pinecone.io/)

An AI-powered Telegram chatbot for Panicker's Travel that provides instant, accurate answers to user queries about tour packages. The chatbot uses a Retrieval-Augmented Generation (RAG) architecture, ensuring that its responses are based solely on the official information provided in the travel brochure.

---

## 🤖 Live Demo

You can interact with the live chatbot on Telegram:

**Telegram Bot Link:** [**https://t.me/paniker_travel_assist_bot**](https://t.me/paniker_travel_assist_bot)

#### Example Interaction:
**You:** `What is the price for the Shimla Manali tour for two people?`

**Bot:** `The price for the Shimla Kufri Manali Rohtang Chandigarh Kurukshetra tour (Tour No. 8) varies by season. During Peak Season (April, May, June, Oct, 20th-31st Dec 2025), the Standard Twin Sharing rate is ₹21,000/- per person, and the Deluxe Twin Sharing is ₹27,500/- per person. Please note this pricing is for a 6 Days/5 Nights tour.`

**You:** `Any Foreign tours available` ?

**Bot:** `Yes, Panicker's Travel offers a selection of tours that include both domestic and a few international (foreign) tours. Please note:`

- `Some of the international tours have dynamic pricing based on the season, hotel category, and group size.`
- `If the exact tour price is not published, it will be marked as "On request."`
- `It is important to confirm the final price directly with Panicker's Travel before booking.`

---

## 📋 Project Overview

This project consists of two distinct n8n workflows that create a powerful, scalable Travel Assistant:

1.  **Knowledge Base Ingestion:** An automated pipeline that watches a Google Drive folder. When a new travel brochure (PDF) is uploaded, it automatically processes the document, creates vector embeddings, and stores them in a Pinecone vector database.
2.  **Telegram Chatbot:** A real-time workflow that listens for messages on Telegram. It uses an AI agent to understand the user's query, retrieves relevant information from the Pinecone database, and generates a precise answer using an OpenAI language model.

### Features
- **24/7 Availability:** The chatbot is always online to assist users on Telegram.
- **Context-Aware:** Leverages RAG to provide answers strictly from the provided brochure, preventing hallucinations.
- **Natural Language Understanding:** Users can ask questions in a conversational manner.
- **Scalable Knowledge:** To update the chatbot's knowledge, simply replace the PDF file in the connected Google Drive folder. The ingestion workflow handles the rest automatically.

---

## ⚙️ How It Works & Technical Details

The architecture is split into two automated workflows powered by n8n.

### 1. Knowledge Base Ingestion (`Vector Creation Travel Assist.json`)

This workflow populates our Pinecone vector database.

**Flow:**
`Google Drive Trigger` → `Download File` → `Document Loader & Splitter` → `OpenAI Embeddings` → `Pinecone Vector Store`

- **Google Drive Trigger:** Activates when a new PDF is added to a specific folder.
- **Document Loader & Splitter:** The downloaded PDF is loaded and split into smaller, overlapping chunks of text to ensure semantic context is preserved.
- **Embeddings OpenAI:** Each text chunk is converted into a numerical vector representation (embedding) using OpenAI's `text-embedding-ada-002` model (1536 dimensions).
- **Pinecone Vector Store:** The generated vectors and their corresponding text are stored (inserted) into a Pinecone index named `legal`.

### 2. Chat Interaction (`Travel Assist.json`)

This workflow handles the live chat with users.

**Flow:**
`Telegram Trigger` → `AI Agent (with Pinecone as a Tool)` → `OpenAI LLM` → `Send Telegram Message`

- **Telegram Trigger:** Listens for incoming messages from the Telegram bot.
- **AI Agent:** An intelligent agent that orchestrates the response.
    - **Tool:** It is equipped with a Pinecone retrieval tool. When asked a question, it uses the user's query to search the `legal` index for the most relevant text chunks from the brochure.
    - **Language Model:** It uses **OpenAI's `gpt-4.1`** model. The retrieved text chunks are passed to the model as context along with the user's original question.
    - **System Prompt:** The agent is instructed to answer *only* from the provided context, ensuring factual accuracy.
- **Send Telegram Message:** The final, context-aware answer is sent back to the user on Telegram.

---

## 📂 Repository Files

* `Panicker_Travel_Brochure.pdf`: The primary knowledge base document for the chatbot.
* `Vector Creation Travel Assist.json`: The n8n workflow for the knowledge base ingestion pipeline.
* `Travel Assist.json`: The n8n workflow for the interactive Telegram chatbot.
* `README.md`: This documentation file.

---

## 🛠️ Setup and Installation

To deploy this project, you will need:
1.  An active **n8n** instance (cloud or self-hosted).
2.  API credentials for:
    * **Telegram Bot**
    * **OpenAI**
    * **Pinecone**
    * **Google Drive** (OAuth2)
3.  A **Pinecone** index created with the name `legal` and `1536` dimensions.
4.  Import both `Vector Creation Travel Assist.json` and `Travel Assist.json` workflows into your n8n canvas.
5.  Configure the nodes in both workflows with your respective credentials.
6.  Activate the workflows.
7.  To begin the ingestion process, upload the `Panicker_Travel_Brochure.pdf` to the Google Drive folder specified in the `Google Drive Trigger` node.
