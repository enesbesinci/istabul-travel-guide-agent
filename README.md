# Istanbul Travel Guide Chatbot with RAG Using Agent from LangGraph

## Introduction:
Hello everyone, in this project we'll develop a Retreival Augmented Generation workflow using Agents from LangGraph.
This project will help you to understand the basic of Adaptive RAG and Agents in LangGraph and also provide a real world example of Agents.

## Goal of The Project:
The aim of this project is to improve the travelling experience of tourists who visiting Istanbul by providing general and up-to-date information about the city.
With this application, tourists will be able to get answers to general questions such as ‘Where is Hagia Sophia Mosque?’ or ‘What should I eat in Eminönü?’ as well as up-to-date
questions such as ‘What are the bus ticket fares in Istanbul?’ or ‘What is the weather like in Istanbul?

## Tech Stack:
* OpenAI for LLM and Embeddings (gpt-4o-mini for LLM and text-embedding-3-small for Embeddings)
* LangGraph and LangChain for creating Agent and RAG workflow
* Chroma for VectorStore
* Tavily for online-searching
* Gradio for user interface

## Note:
This guide assumes that you already have information about the LLM's, RAG and AGENTS

## Dataset:
For this PoC project we'll use 3 guides about Istanbul on the internet to use it. Actually there are more guides to Istanbul on the internet, but after a quick review I decided to use these books.
These books give very detailed and comprehensive information about Istanbul. You can see these guides in the repository.

## Coding:

Firstly, if you are using an LLM and Embedding model from a provider (OpenAI, Google, Anthropic, Mistral etc.) via the API, make sure you have set up your environment variable.
And also we will be using Tavily (a search engine for LLM's) as a tool so to use Tavily you should have a API_KEY. They have a free-trial version!

[Clich here to get a Tavily API_KEY](https://tavily.com/)

Yes, we can build it now.

In this project we will have two source to answer the tourists' question.

## Create an index for the documents:

We create a vectorstore for the documents that we will use. Vectorstores enable semantic search by allowing us to find relevant documents based on the meaning of the query rather than
exact matches. This approach enhances the relevance and accuracy of search results, making it easier for tourists to get the information they need. And also in this project I used the Chroma for vectorstore and "text-embedding-3-small" embeeding model from OpenAI. But there are also different options.
You can use local LLMs or different vectorstores in your project.

For this PoC project we have 3 guides about Istanbul in PDF format. Let's split them up and put them into a vectorstore and create a retriever.

![image](https://github.com/user-attachments/assets/615774e8-d7ab-4d8b-a033-b84cc6c26322)

Let's try the vectorstore by providing a query.

![image](https://github.com/user-attachments/assets/d52a6a8e-7c28-4061-bc94-6c77d1ed56b9)

## Create a Tavily Web Search tool for online searching.





