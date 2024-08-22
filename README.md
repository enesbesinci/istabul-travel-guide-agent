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

We create a web search tool, which is our second source to access current , to use it in Agent.

![image](https://github.com/user-attachments/assets/a02c5c25-b4da-4e81-a154-70cd4dddc403)

Let's check if it's working.

![image](https://github.com/user-attachments/assets/291c4668-2ddb-49d9-8f85-b5462a88dede)

The results look good. So far, we have created two data sources required for the application. Now we can create our nodes and edges that is the most important components of the agent.

## Nodes and Edges: Key Components of the Agent

In this section, we will define various functions. Some of these functions will act as nodes, and others will serve as edges.

* Nodes: These represent individual processing units or operations within the agent. Each node performs a specific task, such as handling a query, performing a search, or generating a response.

* Edges: These define the connections between nodes, determining the flow of information within the agent. An edge represents the transition from one node to the next, allowing for the sequential execution of tasks.


### TranslateQuery:

The first node we'll create will translate the user's question into English. We have two reasons for doing this.

* Documents (guides about Istanbul) that we use in the project is in English Language.
* The language model we will use gives more successful results in English language compared to other languages.
* English is the most populer language in the world so lots of tourist can use this app easliy.

Let's create the first function.

We use an LLM to translate the query/question into English. To do this, we create a prompt that tells the LLM what to do.

#### Promt > Translate the question into English if the question is in another language. \n If the question is in English, do nothing.

After creating Promp, we put Prompt, LLM and StrOutputParser() functions into a chain.

![image](https://github.com/user-attachments/assets/fb9b943f-19fa-417f-9b9b-d379a30b92c7)

Let's check if the node called TranslateQuery is working.

![image](https://github.com/user-attachments/assets/525476fb-77b3-4900-9e66-2503a7749439)

As you can see above, we have passed two questions in different languages to the node. The node called TranslateQuery translated the Turkish question into English. It works well.

### Router:

Once the user's query has been translated into English, we define a function called RouteQuery. This function analyses the user query and routes it to the most relevant data source according to the content of the query.

There are two available data sources in this project:

* Web-Search
* Retriever

Let us analyse this function in more depth. This function is used as a ‘conditional edge’. The function will give a structured output and will return the values contained in the Literal keyword, i.e. either ‘vectorstore’ or ‘web_search’.

#### Prompt > You are an expert at routing a user question to a vectorstore or web search. Vectorstore contains documents about the history of Istanbul, touristic and historical places of Istanbul, and food and travel tips for tourists. Use vectorstore for questions on these topics. If the question is about transportation, weather, and other things, use web search.

![image](https://github.com/user-attachments/assets/e3238b65-8953-4e18-b7a3-7ab78922b6e5)

Let's try the function that we created for routing.

![image](https://github.com/user-attachments/assets/2932e36e-aec6-4a69-a240-2ea5b027b3dd)

The router works well, the first question was about the current events and the second one was about general information about Istanbul so Router routed the questions correctly.

#### Note that the output of the Router function. Router can just give 2 different output.

### Retrieval Grader:

After retrieving the data related to the user query/question from the retriever, we add an additional layer to determine how relevant the retrieved data is to the question.

#### Prompt > You are a grader assessing relevance of a retrieved document to a user question. \n If the document contains keyword(s) or semantic meaning related to the user question, grade it as relevant. \n It does not need to be a stringent test. The goal is to filter out erroneous retrievals. \n Give a binary score 'yes' or 'no' score to indicate whether the document is relevant to the question.

![image](https://github.com/user-attachments/assets/69d1f710-d18c-4a4e-8fbd-b2251f877acc)

Let's try the Retrieval Grader function by passing two different questions.

![image](https://github.com/user-attachments/assets/816b632d-f665-497d-8720-ea26bf2adc12)

The same documents were passed through the Retrieval Grader function using two distinct questions. For the query about Hagia Sophia, the function returned ‘relevant - yes’, whereas for the query about LeBron James, it returned ‘irrelevant - no’.

### Generate:

After receiving the most relevant data to the user question/query. We are ready to generate an answer. For this unique use-case I have created a different prompt for generating.

[Clich here to see the Prompt I desired for this project](https://smith.langchain.com/prompts/istanbul-guide-rag?organizationId=8a67dd04-70e9-5608-9c6c-f6c4792775c5)

Let's write the full code and ask a question.

![image](https://github.com/user-attachments/assets/329b7591-62ff-42d3-ba88-7995ce01510d)

It looks good, we can skip to the next step.

### Hallucination Grader:

This section introduces a system that evaluates whether a language model’s generated response is based on factual information. The model uses a binary scoring system—either 'yes' or 'no'—to determine if the answer is grounded in the provided facts. The process involves setting up a grader using a specific LLM (language model) and a prompt that compares the generated output against a set of retrieved documents.

#### Prompt > You are a grader assessing whether an LLM generation is grounded in / supported by a set of retrieved facts. \n Give a binary score 'yes' or 'no'. 'Yes' means that the answer is grounded in / supported by the set of facts.

Let us then use the function to assess whether the generated answer is consistent with the real data and to help identify possible hallucinations (i.e., fabricated or unsupported content).

![image](https://github.com/user-attachments/assets/bd640548-91c6-4d70-94af-7b6e84b61745)

As illustrated in the image above, the documents retrieved by the retriever align with the model's output, indicating that the response is grounded in factual data. This confirms that the model is not generating hallucinated or unsupported information.

### Answer Grader:

This section defines a system that evaluates whether a generated response effectively addresses the user's question. Using a binary score—'yes' or 'no'—the grader determines if the answer resolves the query. The process involves setting up a language model (LLM) to generate structured output, then comparing the user's question against the model’s generated answer using a specified prompt.

#### Prompt > You are a grader assessing whether an answer addresses / resolves a question \n Give a binary score 'yes' or 'no'. Yes' means that the answer resolves the question.

Let's pass the answer and the user's question/query generated with the Generate function in the previous step to this function and see the results.

![image](https://github.com/user-attachments/assets/4febfbbe-dc20-4929-97fc-2970bb0eaea0)

The result is ‘yes’. This means that the generated answer answers the user's query/question.

### Question Re-writer

This function optimises user questions/queries to improve their suitability for vector-based retrieval. Using a language model, the function interprets the semantic intent of the input question and formulates a more optimised version. The aim is to improve the efficiency of the retrieval process by providing clearer and more precise questions.

#### Prompt > You a question re-writer that converts an input question to a better version that is optimized \n for vectorstore retrieval. Look at the input and try to reason about the underlying semantic intent / meaning.

Notice that the variable named "question" contains the question "Where is the Hagia Sophia? Let's pass the same question to the function and see what happens.

![image](https://github.com/user-attachments/assets/2c75549d-a478-4ff2-b4f1-9ae299af8fc8)

As you can see in the image above, function optimised the question for vector-based search and rewrote it.

#### New Rewritten Question > "What is the location of the Hagia Sophia?"











