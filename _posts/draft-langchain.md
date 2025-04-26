---
layout: post
title:  "4. LangChain Explained: The Framework Behind the AI Apps"
author: skalavala
image: assets/images/langchain.png
categories: [ RAG, AI, langchain ]
tags: [ rag, llm, langchain ]
featured: false
hidden: false
---

LangChain is a popular open-source framework that use Large Language Models (LLMs) to build AI applications. The typical application is building RAG based applications, multi-agent workflows, and complex prompt engineering. It is a great framework to to orchestrate large language models, tools, and various data sources together in a very loosely coupled but cohesive manner.

The term "Chain" in LangChain refers to its ability to chain or orchestrate steps between language models, data sources, tools...etc. Additionally, LangChain it is not a Large Language Model itself. It is just an open-source lightweight Python package that you can run on your personal device.

Using LangChain you can combine multiple steps such as fetching from database, generate response from LLM, process messages...etc. LangChain mainly consists of the following components:

### Agents
LangChain Agents are one of the core functionalities of LangChain, allowing you to build flexible, decision-making systems that can interact with external tools, data sources, and even other models. They are designed to extend the capabilities of language models by enabling them to make decisions, take actions, and retrieve or manipulate data dynamically. In essence, agents are responsible for taking user input and determining the best course of action based on pre-defined rules, external knowledge, and input from various sources.

#### What Are LangChain Agents?
LangChain Agents are systems that use a language model (LLM) in combination with other tools or resources (APIs, databases, external services, etc.) to perform more complex tasks. They don’t just respond to user inputs, they actively decide how to interact with the world and take actions. These agents can automate workflows, answer questions, interact with external systems, and much more.

#### Key Features of LangChain Agents:
Dynamic Decision-Making: Agents can dynamically decide which tools to use, how to interact with them, and in what order, based on the input they receive. Tools in this case are external components that basically extend the functionality of LLM. LangChain is capable of invoking tool calling/make API calls to the tools that are external to the LLM. For example, If you ask LLM on whats the weather in your homw town, LLM by default doesn't know the weather at your location at a specific time. It needs to go get that information for you, and it leverages "tool calling" for that.

Integration with External Tools: Agents can interface with APIs, databases, files, or any system outside of the language model.

Memory Management: Agents can maintain memory between interactions, enabling more personalized and context-aware interactions.

Execution Flow: Agents can orchestrate the sequence of operations to reach a desired outcome, potentially involving multiple steps, tools, and models.