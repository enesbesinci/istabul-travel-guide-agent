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
For this Proof of Concept (PoC), we selected three publicly available Istanbul travel guides. While numerous guides exist, these were chosen for their comprehensive and detailed information about the city. The selected guides are available in the project repository.

## Coding:

Ensure that your environment variables are correctly configured if you are using an API to access LLM and Embedding models from providers like OpenAI, Google, Anthropic, or Mistral. Additionally, you'll need an API key to integrate Tavily, which offers a free trial version.

[Clich here to get a Tavily API_KEY](https://tavily.com/)

Yes, we can build it now.

In this project, two primary data sources will be used to answer tourists' queries:

* A VectorStore containing information from the selected Istanbul guides.
* The Tavily Web Search Tool, for real-time online searching.

## Creating an Index for the Documents:

A vectorstore is created to store the guides. Vectorstores enable semantic searches by retrieving documents based on the meaning of the query, rather than exact word matches, improving the relevance and accuracy of results. This project uses Chroma for document storage and OpenAI's "text-embedding-3-small" model for embeddings. However, other LLMs and vectorstore solutions can also be used.

For the PoC, three PDF guides about Istanbul are split and stored in a vectorstore to create a retriever for querying.

![image](https://github.com/user-attachments/assets/615774e8-d7ab-4d8b-a033-b84cc6c26322)

Let's try the vectorstore by providing a query.

![image](https://github.com/user-attachments/assets/d52a6a8e-7c28-4061-bc94-6c77d1ed56b9)

## Create a Tavily Web Search tool for online searching:

We create a web search tool, which is our second source, to use it in Agent.

![image](https://github.com/user-attachments/assets/a02c5c25-b4da-4e81-a154-70cd4dddc403)

Let's check if it's working.

![image](https://github.com/user-attachments/assets/291c4668-2ddb-49d9-8f85-b5462a88dede)

The results look good. So far, we have created two data sources required for the application. Now we can create our nodes and edges that is the most important components of the agent.

## Nodes and Edges: Key Components of the Agent

In this section, we will define various functions. Some of these functions will act as nodes, and others will serve as edges.

* Nodes: These represent individual processing units or operations within the agent. Each node performs a specific task, such as handling a query, performing a search, or generating a response.

* Edges: These define the connections between nodes, determining the flow of information within the agent. An edge represents the transition from one node to the next, allowing for the sequential execution of tasks.


### TranslateQuery Node:

The first node that we'll create will translate tourists' question to English. We have two reason to do this.

* Documents (guides about Istanbul) that we use in the project is in English Language.
* The language model we will use gives more successful results in English language compared to other languages.
* English is the most populer language in the world so lots of tourist can use this app easliy.

Let's create the node.

We use an LLM to translate the query/question into English. To do this, we create a prompt that tells the LLM what to do.

#### Promt > Translate the question into English if the question is in another language. \n If the question is in English, do nothing.

With this prompt, we specified what the node will do. Finally, let's create a chain consisting of Prompt-LLM-StrOutputParser.

![image](https://github.com/user-attachments/assets/fb9b943f-19fa-417f-9b9b-d379a30b92c7)

Let's check if the node called TranslateQuery is working.

![image](https://github.com/user-attachments/assets/525476fb-77b3-4900-9e66-2503a7749439)

As you can see above, we have passed two questions in different languages to the node. The node called TranslateQuery translated the Turkish question into English. It works well.

### Router Edge:

Once a user sends a question and it is translated into English, the system analyzes the query to determine the most suitable data source for retrieval.

There are two available data sources:

* Web-Search
* Retriever

To facilitate this process, we implement a query routing mechanism that directs user questions to the appropriate data source based on the query's nature. This decision-making process is powered by an OpenAI LLM model with structured outputs, ensuring that each question is routed effectively to either the vectorstore or web search.



#### Prompt > You are an expert at routing a user question to a vectorstore or web search. Vectorstore contains documents about the history of Istanbul, touristic and historical places of Istanbul, and food and travel tips for tourists. Use vectorstore for questions on these topics. If the question is about transportation, weather, and other things, use web search.

![image](https://github.com/user-attachments/assets/e3238b65-8953-4e18-b7a3-7ab78922b6e5)

Let's try the function that we created.

![image](https://github.com/user-attachments/assets/2932e36e-aec6-4a69-a240-2ea5b027b3dd)

The router works well, the first question was about current events and the second was about general information. Router routed the questions correctly.





