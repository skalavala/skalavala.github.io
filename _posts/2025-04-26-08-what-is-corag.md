---
layout: post
title:  "8. May the CoRAG Be With You: Chaining Retrievals Like a Jedi"
author: skalavala
image: assets/images/corag1.png
categories: [ RAG, AI, corag ]
tags: [ rag, llm, corag ]
featured: true
hidden: false
---

We've talked about RAG, Multi-Stage and Condensed RAG. Now its time to go one step further, which is CoRAG. Let's see what this CoRAG is all about, and how it differentiate from other RAG implementations we talked about in the past.

RAG (Retrieval Augmented Generation) allows us to feed our own data (documents, files, web pages) to a language model, enabling it to generate more accurate and contextually relevant responses. It bridges the gap between static pre-trained models and dynamic, real-world applications by incorporating external knowledge into the generation process.

CoRAG, or Chained Retrieval Augmented Generation, takes this concept further by enabling a sequence of retrievals and generations. Instead of relying on a single retrieval step, CoRAG chains multiple retrievals together, allowing for deeper contextual understanding and more complex reasoning. This approach is particularly useful when dealing with multi-step problems or when the required information is distributed across multiple sources.

In the following sections, we will explore how CoRAG works, its advantages over traditional RAG, and practical use cases where it shines.

### What is CoRAG and How does it compare to traditional RAG?

CoRAG is not just about finding documents that match some keywords. It is about understanding the situation and retrieving information dynamically based on the evolving context of the interaction.

Instead of treating every question in isolation, CoRAG systems look at the whole conversation, the user's intent, and the background knowledge that is already on the table. Then they decide what information would actually help move things forward.

Traditional RAG is like asking for directions and someone handing you a full city map without saying anything. You are left staring at a hundred street names trying to figure it out yourself. 

CoRAG is like asking for directions and someone walking over, looking at where you are trying to go, and drawing you a simple path right on the map. It is not just about giving you some information, it is about giving you the information that actually helps you move forward.

In a classic RAG setup, the retrieval step is usually very basic. It takes the user’s latest question, turns it into a vector, searches a database, and grabs the top few results. Simple and fast, but it doesn't see the bigger picture.

If you ask a vague follow-up question like "What about pricing?", a regular RAG system might have no clue what "pricing" refers to, because it lost the thread of the conversation. It pulls random pricing information from the database, not realizing you were asking about pricing for a specific software you mentioned two questions ago.

CoRAG fixes this by making retrieval aware of context. It uses more than just the last sentence. It can look at the full chat history, previous retrievals, memory, and can connect and build a better context by looking into all previous questions. Let me give you an example:

When a user asks a question, "I am planning a trip to New York", and then later asks, "what is the best time to visit New York", and later "are the trains expensive?", the user did not directly ask "Help me plan an affordable trip to New York during best time of the year." It understands what the user is trying to do, and can help provide more relevant answers instead of returning a bunch of articles about trains, deals, or times.

If you remember from our previous posts, here is the sequence of steps that goes into the Traditional RAG.

<center>
<img src="{{ site.baseurl }}/assets/images/rag-steps.png" alt="Traditonal RAG Steps"/>
</center>
<br/>
In the CoRAG, the steps are more like this:
<center>
<img src="{{ site.baseurl }}/assets/images/corag-steps.png" alt="CoRAG Steps"/>
</center>
<br/>
I've highlighted the differences between Traditional RAG and Chain of RAG, in the images above, so that it is clear for you. As you see in the CoRAG, you do a ittle more work ahead in terms of preparing the context, and even utlizes Condensed RAG techniques we talked about in the last post.  

If you think about it, Traditional RAG and CoRAG are a bit like Traditional IRA and Roth IRA.
In a Traditional IRA, you save first and deal with taxes later, which feels efficient upfront but can cause surprises down the road. In the same way, Traditional RAG retrieves blindly based on the latest input and hopes everything lines up later in the conversation.

CoRAG however, like a Roth IRA, plans smarter from the start. It does a little more work upfront by gathering context and understanding goals, so that when it is time to answer, everything flows more smoothly with fewer unexpected problems.

If you are already using frameworks like `LangChain`, `LlamaIndex`, you are in luck, as they support the concept of chaining, and you can implement CoRAG natively using these frameworks.

## Why should you care?
CoRAG is a natural extension to the Traditional RAG, Multi-Stage/Condensed RAG, GraphRAG, and more. As each RAG implementation evolved to solve specific problems like relevance, scalability, and context compression, CoRAG takes it a step further by making retrieval fully aware of the conversation's history, intent, and dynamic goals. <i>It is not just about pulling better chunks or chaining steps together</i>, it is about building retrieval systems that can think, adapt, and retrieve exactly what the user needs at every point in the interaction.

Is CoRAG the silver bullet for all RAG based implementations? I don't think so! There will be many more combinations of RAG approaches in the future, each tuned for different types of problems, industries, and user needs. CoRAG solves a big piece of the puzzle by making retrieval smarter and more adaptive, but there will always be room for new strategies, whether that is hybrid retrieval methods, deeper graph-based reasoning, or even agent-driven retrieval workflows. The important thing is that we are moving closer to building AI systems that do not just retrieve information, but understand how to retrieve the right information at the right time.

CoRAG is not about swinging a lightsaber wildly and hoping for the best. It is about using the Force wisely, chaining your retrievals with skill, and knowing when to trust your context. May the force be with you!

References:

1. Microsoft has published [A public repo for CoRAG here](https://github.com/microsoft/LMOps/tree/main/corag)
2. Want to go even deeper? [Read scientific paper here](https://arxiv.org/pdf/2501.14342v2)
