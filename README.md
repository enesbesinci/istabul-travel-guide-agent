# Travel Guide Chatbot with RAG Using Agent from LangGraph

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

### Question Re-writer:

This function optimises user questions/queries to improve their suitability for vector-based retrieval. Using a language model, the function interprets the semantic intent of the input question and formulates a more optimised version. The aim is to improve the efficiency of the retrieval process by providing clearer and more precise questions.

#### Prompt > You a question re-writer that converts an input question to a better version that is optimized \n for vectorstore retrieval. Look at the input and try to reason about the underlying semantic intent / meaning.

Notice that the variable named "question" contains the question "Where is the Hagia Sophia? Let's pass the same question to the function and see what happens.

![image](https://github.com/user-attachments/assets/2c75549d-a478-4ff2-b4f1-9ae299af8fc8)

As you can see in the image above, function optimised the question for vector-based search and rewrote it.

#### New Rewritten Question > "What is the location of the Hagia Sophia?"

### Output Guardrails:

This function checks whether the answer contains negative content for users.

This function checks whether the answer contains negative content for users.

![image](https://github.com/user-attachments/assets/2f454fef-ab97-41d4-98c3-49393d4d2c86)

Let's check the function.

![image](https://github.com/user-attachments/assets/730b8607-5d7e-40dd-8340-011e20d4cb4e)

We have created all the funcstions to use it in the Agent. We'll create nodes and edges using these functions. Now we can skip to the next step.

## GraphState Defination:

The first thing we should do when define a graph is define the State of the graph. Now we'll define a State and specify the parameters that it will take. For this RAG flow we have 3 parameters.

* question > User question/query
* generation > LLM generation
* documents > List of documents

![image](https://github.com/user-attachments/assets/53381d1a-ca7b-4c00-96f2-fd79cec817f2)

After creating the State we can create the most important components of the an Agent. Nodes and Edges.

## Creating Nodes and Edges:

Let's start by creating nodes.

In short, Nodes receive the current State as input and return a value that updates that state.

### Retrieve Node:

![image](https://github.com/user-attachments/assets/049e6067-aa01-493a-8b42-727ba661f809)

Retrieve Node takes the question of the State as input and returns the relevant documents to the question as output.
As you can in the image above, we have used the "retriever" function we have created before.


### Translator Node:

![image](https://github.com/user-attachments/assets/878a106a-0311-4aa7-8db3-12a9cb9ca56a)

The Translator node takes the State question as input and translates the query using the question_translater’ function, then outputs the translated question as output.

### Generate Node:

![image](https://github.com/user-attachments/assets/fda43410-d879-4c50-af09-c0855051f752)

Generate Node takes the question and a list of documents of the State as input and returns a response as output. It does this with the function we defined called ‘rag_chain’.


### Grade Documents:

![image](https://github.com/user-attachments/assets/3f1321c6-c296-46d6-9cc4-8c18d8532569)

Grade Documents Node takes question and documents of the State and  evaluates using the retrieval_grader function and returns only the relevant documents.


### Transform Query Node:

![image](https://github.com/user-attachments/assets/cee8ff65-5680-464e-b999-12929a83d394)

Transform Query Node transforms the query to produce a better the question by using question_rewriter functions that we defined above.


### Web Search Node:

![image](https://github.com/user-attachments/assets/39c72efe-af7d-40b1-832e-df63abf7f81e)

### Output Guardrails Node:

As you know, this application will be used by people from many different cultures, countries, races and genders. Therefore, we want to filter answers that contain bad/inappropriate content about these people.

Finally, we will add a Node that takes the generated output as input. If the generated output contains any content that is racist, sexist or against human rights, we do not want to show the answer to the user.

#### That's it. We have defined the all of the Nodes of the Agent. Now we can continue by defining the Edges.

![image](https://github.com/user-attachments/assets/a5be99aa-d4f9-4428-b362-b3cb8cfad9e8)

### Route Question Edge:

![image](https://github.com/user-attachments/assets/93baa819-5d7a-4506-8d49-9eac82da64d9)

Route Question Edge takes the user question as input and redirects it to ‘web_search’ or ‘vectorstore’ based on the content of the question. The question_router function is used for this.

### Decide To Generate Edge:

![image](https://github.com/user-attachments/assets/d54e032c-4da6-41c8-9431-8328162d5592)

Decide To Generate Edge takes the question and filtered documents, if there are any relevant documents it returns ‘generate’ output to generate the answer, if there are no relevant documents it returns ‘transform_query’ output.

### Grade Generation and Documents and Question Edge:

![image](https://github.com/user-attachments/assets/0063bac8-db15-4b53-b4b1-30d4eb168dd1)

Determines whether the generation is grounded in the document and answers question.

#### We have defined all Nodes and Edges. Now, we can continue by building the Graph.

## Building Graph:

Firstly, we define the nodes in the Graph we created.

![image](https://github.com/user-attachments/assets/a19d29e1-8d73-4bf4-b110-e7f2b6dc7406)

We have defined all the nodes. Now let's create a logic for our Agent and add Edges.

![image](https://github.com/user-attachments/assets/85d1e803-80df-434d-a89c-117746faddbf)

As you can see in the code, we determined the flow of the graph. We did this with the help of various edges. The nodes are simple Python functions that process the input from the edges. This is a good reason to work with LangGraph instead of LangChain agents.

Finally, let's compile the Graph we created. This is a required step.

![image](https://github.com/user-attachments/assets/7e44d4d5-513f-4e0b-8fe6-634202011ad3)

## Visualization of the Agent Flow:

We can visualise Agent Flow for better understanding and error checking.

![image](https://github.com/user-attachments/assets/c6d876d8-4465-4927-a449-3ecdbff8e64e)

Let's see the graph.

![image](https://github.com/user-attachments/assets/e818c0b8-e138-498b-a9bd-57113abd6206)


## Run the Agent:

Now that everything's ready, we can ask the Agent our questions.

Firstly, I will ask a question to use the web-search node.

![image](https://github.com/user-attachments/assets/8f951f07-501b-440d-bd67-0d5bcb23e10e)

The response looks good. Here is the full response:


As of now, the weather in Istanbul is quite pleasant. The current temperature is approximately 28.2°C (82.8°F), and it is sunny, making it a great day to explore the city. The humidity level is around 70%, which can make it feel a bit warmer, with a "feels like" temperature of 28.3°C (83.0°F). \n\nThere is a moderate breeze coming from the west-southwest at about 11.9 mph (19.1 kph), which can provide some relief from the heat. The atmospheric pressure is 1008 mb, and there are no precipitation forecasts, so you can expect a dry day. Visibility is good at 10 km (6 miles), and the UV index is relatively high at 8.0, so it\'s advisable to wear sunscreen if you\'re spending time outdoors.\n\nOverall, it\'s a beautiful sunny day in Istanbul, perfect for sightseeing and enjoying the city\'s rich history and vibrant culture!

Note also that we can see the stages through which the answer is generated.

![image](https://github.com/user-attachments/assets/6cea5982-01ae-4e05-897c-07ffd0331686)

Now let's ask another question where the agent will use the vector-store, but I will ask this question in Turkish.

#### Question: İstanbul'da gezmek için 3 tane tarihi mekan önerebilir misin? Bu mekanlardan çok kısaca bahset?

![image](https://github.com/user-attachments/assets/0f00f42e-0012-491d-b443-f6d045cff402)

#### Response: Istanbul is rich in history and offers numerous historical sites to explore. Here are three must-visit historical places:\n\n1. **Hagia Sophia (Ayasofya)**: Originally constructed as a Christian basilica in 537 AD, Hagia Sophia is renowned for its massive dome and stunning mosaics. It served as a cathedral for nearly a thousand years before being converted into a mosque after the Ottoman conquest of Constantinople in 1453. Today, it stands as a museum, showcasing its architectural grandeur and historical significance.\n\n2. **Topkapı Palace**: This opulent palace was the primary residence of the Ottoman sultans for approximately 400 years. Located in the Historic Peninsula, Topkapı Palace is a sprawling complex that includes beautiful courtyards, exquisite tile work, and a rich collection of artifacts, including the famous Topkapi Dagger and the Prophet Muhammad's cloak. Visitors can explore the various sections of the palace, including the Harem, which offers a glimpse into the private lives of the sultans and their families.\n\n3. **Blue Mosque (Sultan Ahmed Mosque)**: Known for its striking blue tiles that adorn its interior, the Blue Mosque is an iconic symbol of Istanbul. Built between 1609 and 1616, it features six minarets and a grand courtyard. The mosque is still an active place of worship, so visitors are encouraged to dress modestly and respect prayer times. Its stunning architecture and serene atmosphere make it a must-see for anyone visiting the city.\n\nThese sites not only reflect Istanbul's rich history but also its cultural and architectural diversity, making them essential stops on your journey through the city.

Now let's ask Agent a question in Korean.

![image](https://github.com/user-attachments/assets/31408a29-124f-4d84-a7fa-65d63142c8de)

#### Question: 쉴레이마니예 모스크의 역사에 대한 정보를 제공하세요. (Provide some information about the history of the Sheleimaniye Mosque)

#### Response: The Suleymaniye Mosque is one of the most significant landmarks in Istanbul, both architecturally and historically. It was commissioned by Sultan Suleyman the Magnificent, one of the most illustrious rulers of the Ottoman Empire, and was completed in 1557. The mosque was designed by the renowned Ottoman chief architect Mimar Sinan, who is celebrated for his contributions to Ottoman architecture and is often regarded as one of the greatest architects in history.\n\nThe mosque is situated on one of the highest hills in Istanbul, allowing it to dominate the city's skyline. It was built as a grand statement of the Ottoman Empire's power and sophistication, serving as an Islamic counterpart to the Byzantine Hagia Sophia, which had been a symbol of Christian architecture for centuries.\n\nThe architectural design of the Suleymaniye Mosque is particularly noteworthy for its vast central dome, which is surrounded by tiered domes, creating a harmonious and majestic interior space. Sinan's innovative use of the dome was a significant advancement in architectural design, and it is often cited as a perfect example of his mastery in this area.\n\nIn addition to its architectural beauty, the mosque complex includes a variety of other structures, such as a madrasa (educational institution), a hospital, and a public kitchen, which served the poor. The Suleymaniye Mosque not only serves as a place of worship but also as a center for community life in Istanbul.\n\nOverall, the Suleymaniye Mosque stands as a testament to the artistic and cultural achievements of the Ottoman Empire during its zenith, reflecting the empire's commitment to both religious devotion and architectural innovation.

## Conclusion:

In this project, we developed a Travel Guide Chatbot using Retrieval-Augmented Generation (RAG) with Agents from LangGraph. The chatbot integrates multiple technologies such as OpenAI's LLM for embeddings, Chroma for document storage, and Tavily for online search capabilities. By combining vector-based retrieval with real-time search, the chatbot effectively addresses various tourist queries, from historical landmarks to current weather conditions.

The implementation demonstrates how nodes and edges can structure an agent's decision-making process, ensuring accurate and relevant responses. This approach enhances tourists' experience by providing comprehensive, up-to-date information about Istanbul in a user-friendly manner. The project showcases the power of adaptive RAG workflows in creating efficient and interactive tools for real-world applications.












