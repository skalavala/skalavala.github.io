---
layout: post
title:  "4. The RAG Awakens: A New Stage in Retrieval"
author: skalavala
image: assets/images/multi-stage-rag.png
categories: [ RAG, AI, embedding, indexing, reranking ]
tags: [rag, llm, chunking, embedding, indexing, reranking]
featured: true
hidden: false
---

In this post, we'll discuss about Multi-Stage/Multi-Hop RAG, or Condensed RAG. Before we go further, let's quickly recap on the traditional RAG. 

In the traditional RAG pipeline, the process begins by ingesting a set of documents, which are then broken down into smaller chunks using various chunking strategies (we discussed them before). These chunks are transformed into embeddings and stored in a vector database, where they are indexed and ready for retrieval.

When a user submits a query, it is converted into an embedding as well, allowing the system to search for relevant documents based on the similarity of the query embedding to the document embeddings.

The retrieved documents are then ranked according to their relevance to the query. This set of relevant documents, along with the query embedding, is used to create the context for the generative model. 

The context is then passed to the large language model, which processes it to generate a final response, which is subsequently returned to the user. It is essential that we understand the steps/tasks involved in the traditional RAG before diving into the advanced RAG concepts.

The traditional RAG works great for standard queries. But when the query becomes complex, that's when we need to start looking into Multi-Stage RAG, where it can handle complex queries and offers quality responses by allowing a structured approach to each step/task.


### What is a Multi-Stage RAG?

The end goal for RAG is ALWAYS about getting more relevant responses. In order to get quality responses, we need to think through several steps and see how we can optimize them. When you optimize each of the steps, that's when we get quality responses. Let's see how it can be done.

Here are the steps we discussed above:

<center>
<img src="{{ site.baseurl }}/assets/images/rag-steps.png" alt="Traditonal RAG Steps"/>
</center>
<br/>

In the Multi-Stage RAG implementation, instead of sequentially going through each step without quality inspection, we take a step back and evaluate before moving to the next step. We may repeat some steps until we are satisfied with one step before moving to the next step.

Think of a multi-stage RAG pipeline like a factory line of workers, each responsible for a specific task. The first worker chunks and embeds documents, hands them to the second worker who indexes them, and so on, each step performing a well-defined operation before passing the result down the line. Eventually, the final worker (the LLM) takes the assembled parts and produces the finished product: an answer.

But here’s the catch... if one of the early workers makes a mistake (say, chunking too broadly or retrieving irrelevant documents), that defect doesn’t get caught immediately. It quietly travels down the line, handed off from worker to worker, and by the time it reaches the last stage, the output might look polished, but it’s built on flawed parts. Garbage in, garbage out, but with extra confidence and formatting.

That’s why, in a robust multi-stage RAG setup, you don’t just automate the handoffs, you also need quality control. Each stage should not only do its job but verify the quality of the input it received. Otherwise, you’re just amplifying the assumptions, errors, made earlier in the pipeline.

Multi-Stage RAG (also known as Condensed RAG) really shines when you throw it the big questions, the kind that are ambiguous, require some reasoning, or need to pull from multiple knowledge areas at once. Think of it as RAG with a bit more brainpower and skill.

For example, when you ask ChatGPT something like “What are the top trends in IT this year?”, it's not just grabbing a few chunks and responding. Behind the scenes, it’s likely using a multi-stage process: retrieving tons of data, ranking it, and reranking it, filtering out the noise, and surfacing the most relevant insights...etc. If it used basic RAG, it might’ve just dumped a bunch of text (which may not be relevant). But with multi-stage RAG, it’s more like a curated content, something that is on-point.

<center>
<img src="{{ site.baseurl }}/assets/images/multi-stage-rag-flow.png" alt="Multi-Stage RAG" width="300"/>
</center>
Here is a sample on how you can implement Multi-Stage RAG. Due to the use of LLMs, you would need OPEN_API_KEY. Make sure you either hardcode the key or pass it through commandline environment variable.

```python
from langchain.embeddings import OpenAIEmbeddings
from langchain.vectorstores import FAISS
from langchain.chat_models import ChatOpenAI
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.prompts import PromptTemplate
from langchain.schema import Document
from langchain.output_parsers import StrOutputParser
from langchain.chains.combine_documents.stuff import StuffDocumentsChain
from langchain.chains import LLMChain
import os

os.environ["OPENAI_API_KEY"] = "sk-..."

# Let's use some of our favorite hobby topics as input :)
docs = [
    Document(page_content="Artificial intelligence is transforming the world. It powers everything from chatbots to autonomous vehicles."),
    Document(page_content="Smart home devices like thermostats and lights can be controlled using voice assistants."),
    Document(page_content="3D printing is revolutionizing manufacturing by enabling rapid prototyping."),
    Document(page_content="Drones are being used in agriculture, delivery, and even cinematography."),
    Document(page_content="A home lab is a personal data center where enthusiasts experiment with servers, networks, and virtualization."),
]

# chunking
splitter = RecursiveCharacterTextSplitter(chunk_size=200, chunk_overlap=50)
chunks = splitter.split_documents(docs)

# get the embeddings
embeddings = OpenAIEmbeddings()
vectorstore = FAISS.from_documents(chunks, embeddings)
retriever = vectorstore.as_retriever(search_type="similarity", search_kwargs={"k": 6})

# question condensation
condense_prompt = PromptTemplate.from_template(
    """You are a helpful assistant. Given the following relevant text snippets, pick the top 3 most useful ones for answering the question: "{question}"

Snippets:
{context}

Return the top 3 most relevant snippets."""
)

llm = ChatOpenAI(temperature=0)
reranker = LLMChain(llm=llm, prompt=condense_prompt, output_parser=StrOutputParser())

qa_prompt = PromptTemplate.from_template(
    """Answer the following question based on the provided context:

Context:
{context}

Question: {question}

Answer:"""
)

qa_chain = StuffDocumentsChain(
    llm_chain=LLMChain(llm=llm, prompt=qa_prompt),
    document_variable_name="context"
)

# full multi-stage pipeline
def multi_stage_rag_pipeline(query: str):
    retrieved = retriever.get_relevant_documents(query)
    context_blob = "\n\n".join([doc.page_content for doc in retrieved])
    condensed = reranker.invoke({"context": context_blob, "question": query})
    final_context_docs = [Document(page_content=chunk) for chunk in condensed.split("\n") if chunk.strip()]
    return qa_chain.run(final_context_docs, question=query)

# example query
response = multi_stage_rag_pipeline("How are drones used in different industries?")
print(response)

```
The Multi-Stage RAG code abve implements a two-step retrieval and response process.

Initial Retrieval (Stage 1): It takes a user question and uses a vector similarity search on embedded documents to fetch a bunch of possibly relevant chunks.

Condensation + Second Pass (Stage 2): Instead of sending all those chunks to the LLM directly, it summarizes or condenses them into a more focused piece of context. This refined context is then sent along with the original question to the LLM to generate a more accurate, thoughtful answer.

Instead of dumping a pile of raw info on the LLM (like traditional RAG does), this approach distills it first, like organizing messy notes into well organized summary notes before asking the model to answer.

I hope by now you've gained a clear understanding of how to effectively approach RAG in practical applications.