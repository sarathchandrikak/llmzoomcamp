# Building a RAG-Based FAQ Assistant with LLMs

## Introduction

Large Language Models (LLMs) have changed the way we build applications. Instead of creating complex rule-based systems, we can now leverage powerful pre-trained models to understand and generate human-like text.

In this project, we will build a **Course FAQ Assistant** using **Retrieval-Augmented Generation (RAG)**. The assistant will answer questions about a course using internal course materials that were not part of the LLM's original training data.

The objective is to understand:

* What LLMs are
* How RAG works
* How search and retrieval systems support LLMs
* How to build a production-style FAQ assistant
* How Agentic RAG extends traditional RAG systems

---

# Understanding Large Language Models (LLMs)

## What is an LLM?

A Large Language Model is a machine learning model trained on massive amounts of text data.

Its core capability is simple:

> Given some text, predict what comes next.

Example:

Input:

```text
The capital of France is
```

Output:

```text
Paris
```

The model repeatedly predicts the next token (word or subword) until a complete response is generated.

---

## Treat LLMs as a Black Box

When building applications, we do not need to understand:

* Transformer architecture
* Attention mechanisms
* Neural network internals
* Training procedures

Instead, we can treat the model as a black box:

```text
Input Text
      ↓
     LLM
      ↓
Generated Response
```

Our focus is:

* Building applications around LLMs
* Supplying relevant information
* Designing prompts
* Creating user experiences

---

# The Problem with LLMs

LLMs are trained on historical datasets.

They do not automatically know:

* Internal company documents
* Private knowledge bases
* Latest course material
* Proprietary business data

Example:

Suppose a course was created last week.

The LLM has never seen it during training.

Therefore it cannot accurately answer questions about it.

This is where RAG becomes useful.

---

# What is Retrieval-Augmented Generation (RAG)?

RAG is a technique that combines:

1. Information Retrieval
2. Large Language Models

The goal is to provide the model with relevant information at runtime.

Instead of relying solely on training data, the model receives:

* User question
* Relevant documents
* Instructions

Then it generates an answer.

---

## Traditional LLM Flow

```text
User Question
      ↓
     LLM
      ↓
Answer
```

Knowledge is limited to training data.

---

## RAG Flow

```text
User Question
      ↓
Search System
      ↓
Relevant Documents
      ↓
LLM
      ↓
Answer
```

The answer is now grounded in our own data.

---

# Real-Life Example

## Course FAQ Assistant

Suppose we have:

* Course syllabus
* Assignments
* Lecture notes
* FAQ documents
* Course policies

A student asks:

```text
Can I submit Assignment 3 late?
```

The system:

1. Searches course documents
2. Retrieves assignment policy
3. Sends policy to LLM
4. Generates final answer

The answer is based on actual course information.

---

# Why Not Send All Documents?

A naive solution would be:

```text
All Documents
      ↓
     LLM
```

Problems:

### Expensive

LLMs charge based on tokens.

More documents = Higher cost.

### Slow

Larger prompts increase latency.

### Context Window Limits

Every LLM has a maximum number of tokens it can process.

Large document collections exceed these limits.

Therefore we only send the most relevant information.

---

# Search and Retrieval

Retrieval is the heart of a RAG system.

The goal is:

> Find the most relevant pieces of information for a user's question.

---

## Search Index

Instead of scanning every document every time:

```text
Question
      ↓
1000 Documents
      ↓
Search
```

We build an index.

```text
Documents
      ↓
Index
      ↓
Fast Search
```

This significantly improves performance.

---

# Search Engines Used in RAG

Several search libraries can be used.

## Apache Lucene

Popular search engine library.

Features:

* Full-text search
* Ranking
* Boosting
* High performance

---

## Elasticsearch

Built on top of Lucene.

Provides:

* Distributed search
* Scalability
* REST APIs
* Production-grade deployments

---

## Apache Solr

Another Lucene-based platform.

Provides:

* Search APIs
* Analytics
* Clustering

---

## MiniSearch

Lightweight search engine.

Useful for:

* Small projects
* Educational projects
* Local experimentation

For a course FAQ dataset, MiniSearch is often sufficient.

---

# Field Boosting

Not all fields are equally important.

Consider:

```json
{
  "question": "How do I submit assignments?",
  "answer": "Assignments are submitted through Canvas."
}
```

User query:

```text
submit assignments
```

The match in the question field is more important.

We can boost fields.

Example:

```javascript
boost: {
  question: 2,
  answer: 1
}
```

Meaning:

* Question is twice as important
* Answer has normal importance

---

## Boost Values

### Greater Than 1

```text
2
3
5
```

More important.

---

### Equal to 1

Default importance.

---

### Less Than 1

```text
0.5
0.2
```

Less important.

---

# Core Components of RAG

A RAG pipeline consists of three major stages.

---

## 1. Retrieval

Find relevant documents.

Example:

```text
Question:
Can I submit late assignments?
```

Retrieve:

```text
Students may submit assignments
up to 48 hours late.
```

---

## 2. Prompt Construction

Build the prompt.

Prompt typically contains:

### Instructions

Fixed section.

Example:

```text
You are a course assistant.
Answer only using the provided context.
```

---

### User Prompt Template

Dynamic section.

Example:

```text
Question:
{question}

Context:
{retrieved_documents}
```

This changes for every request.

---

## 3. LLM Generation

Final prompt is sent to the model.

```text
Prompt
     ↓
    LLM
     ↓
Answer
```

---

# System Prompt

Every LLM application typically has a hidden system prompt.

Example:

```text
You are a helpful teaching assistant.
Always answer politely.
Use only provided context.
```

The user never sees this prompt.

It controls model behavior.

---

# RAG Architecture

The application generally contains two independent processes.

---

## Process 1: Data Ingestion

Purpose:

Convert documents into a searchable index.

### Steps

```text
Documents
      ↓
Cleaning
      ↓
Chunking
      ↓
Indexing
      ↓
Search Database
```

This process runs periodically.

Often called:

* Ingestion Pipeline
* Indexing Pipeline
* ETL Pipeline

---

## Process 2: RAG Assistant

Purpose:

Answer user questions.

### Steps

```text
User Question
       ↓
Search
       ↓
Retrieve Context
       ↓
Build Prompt
       ↓
LLM
       ↓
Response
```

---

# Traditional RAG Limitations

Traditional RAG follows a fixed workflow.

```text
Search
   ↓
Retrieve
   ↓
Prompt
   ↓
LLM
```

The flow is rigid.

Capabilities are limited.

Example:

```text
What is the refund policy?
```

Works well.

---

But what if the user asks:

```text
Compare refund policies and assignment policies.
```

Or:

```text
Find all references to deadlines.
```

Traditional RAG may struggle.

---

# Introduction to Agents

Agents provide decision-making capabilities.

Instead of following a fixed workflow:

```text
Search
   ↓
LLM
```

The LLM decides:

* What to search
* When to search
* How many searches to perform
* What information to retrieve

---

# Agentic RAG

Agentic RAG combines:

* Retrieval
* Search Tools
* LLM Reasoning

The LLM acts as a controller.

---

## Agent Workflow

```text
User Question
       ↓
Agent
       ↓
Search Tool
       ↓
Retrieved Data
       ↓
Agent
       ↓
LLM Response
```

---

## Example

Student asks:

```text
What topics are covered in Module 3?
```

Agent decides:

1. Search course modules
2. Retrieve Module 3 content
3. Retrieve related assignments
4. Combine information
5. Generate answer

---

# Traditional RAG vs Agentic RAG

| Traditional RAG       | Agentic RAG              |
| --------------------- | ------------------------ |
| Fixed workflow        | Dynamic workflow         |
| Single retrieval step | Multiple retrieval steps |
| Limited reasoning     | Advanced reasoning       |
| Simple implementation | More powerful            |
| Best for FAQs         | Best for complex tasks   |

---

# Project Goal

Build a Course FAQ Assistant capable of:

* Ingesting course documents
* Creating a search index
* Retrieving relevant information
* Building prompts dynamically
* Answering student questions
* Exploring Agentic RAG capabilities

---

# End-to-End Architecture

```text
                 COURSE DOCUMENTS
                         │
                         ▼
                 INGESTION PIPELINE
                         │
                         ▼
                    SEARCH INDEX
                         │
                         ▼

Student Question ──► Retrieval Engine
                         │
                         ▼
                Relevant Context
                         │
                         ▼
                  Prompt Builder
                         │
                         ▼
                         LLM
                         │
                         ▼
                 Final Answer
```

---

# Key Takeaways

* LLMs generate text by predicting the next token.
* We can treat LLMs as black boxes when building applications.
* RAG allows LLMs to answer questions using external knowledge.
* Search and indexing are critical parts of RAG systems.
* Field boosting improves retrieval quality.
* A RAG system consists of Retrieval → Prompt Building → Generation.
* Traditional RAG is fixed and limited.
* Agentic RAG allows the LLM to control retrieval dynamically.
* FAQ assistants are one of the most common real-world RAG applications.
