---
layout: post
title:  "3. Chunk Like a Pro: Strategies That Actually Make RAG Work."
author: skalavala
image: assets/images/chunking1.png
categories: [ RAG, AI, chunking, embedding, indexing ]
tags: [rag, llm, chunking, embedding, indexing]
rating: 4.5
featured: true
hidden: false
---

When you're building RAG-based applications, one of the first things you'll need to do is break your documents into smaller, more manageable pieces. This process is known as chunking. This is a crucial step that makes the retrieval process faster and more accurate. By slicing large documents into bite-sized chunks, you make it easier for the LLM to find and serve up the most relevant bits of information when a user asks a question.

Think of it like prepping ingredients before you cook. If you toss in a whole onion, don’t be surprised when the dish turns out weird.

Language models have context length limitations (i.e., they can only “see” so many tokens at once), so you can’t just toss in a 100-page PDF and expect magic. That’s where chunking comes in. It’s how we give the model just enough context to be helpful without frying its brain. When you chunk your documents the right way, you get useful, relevant answers. When you don’t? Well, let’s just say garbage in, garbage out. That’s why choosing the <strong>right chunking strategy</strong> isn’t just a technical detail; it’s the secret sauce behind RAG systems that actually sound like they know what they’re talking about.

If you are familiar with how the IDS/IPS systems work in the cybersecurity domain, the following example might make more sense.

You wouldn't just dump the entire network traffic log into the Intrusion Detection System (IDS) engine and hope it finds something useful. You basically segment it, normalize it, and make sure each packet of data is meaningful on its own. Just by looking at that packet, if I can't make sense of where it came from, who generated it, and how it got logged, I basically lost the context of it, and most likely the IDS system will struggle too. In the RAG world, chunking works the same way. You're breaking down massive documents into digestible, semantically relevant chunks so the language model can "inspect" them effectively. If your chunks are too big, the model might miss the threat (or the answer). Too small, and it’s all false positives, a bunch of noise and not enough signal. The right chunking strategy is like fine-tuning your threat rules: precision in, relevance out. Otherwise, garbage in, garbage out!

And just like in the IDS example, bad input equals bad output, only this time, the LLM isn’t just confused, it is giving you wrong answers more confidently. :)

## Before You Chunk - What to Know?

Before you start chunking your documents, it is critical we understand some of the best practices. Doing it wrong may break the performance of the RAG system.

1. Each chunk should be relevant and understandable. If you read a chunk out of context, you should still be able to understand what it is all about. If it feels like a sentence fragment or out-of-context gibberish, the model’s going to be just as confused as you are. 
2. Pick the right chunk size. Based on the example above, too big of a chunk, and you might overflow the model's context window. Remember the context window size is limited, and you don't want to store junk in there. If the chunk size is small, you lose semantic meaning. 

So you might ask, how do I pick the right size? Unfortunately, there is no straightforward answer. It is part science, part art, and part trial and error. You really have to understand your documents well before you inspect the chunks and see if the chunks are done right. You have to play around with the chunk sizes and overlaps until you get to a point where the individual chunks are meaningful and can yield good responses. The overlap is when you break your document into multiple chunks; each chunk should carry some context from the previous to the next. There should be a fine balance on this as well.

### Can I Just Use One Chunking Strategy for All My Documents?

Absolutely not. Every document is different. Legal contracts need different chunking than API docs. Chat transcripts need different treatment than blog posts. Your chunking strategy should match the structure, language, and use case of the content. This is why there are multiple strategies, and why we're about to discuss them next. Let's go one by one from simplest to complex.

### 1. Fixed Size Chunking

Fixed-size chunking is exactly what it sounds like: you break your document into equal-sized blocks of text based on a fixed number of tokens, words, or characters. No additional logic. No structure or context awareness. You basically break the document into multiple sentences without really understanding anything about the document content.

Sadly, this is the "default" chunking strategy that most people use in their applications - mainly because it does the job, gives relevant enough responses (may not be perfect), saves a lot of time, and the team can focus on other things until someone complains about poor quality of responses.

If you are using fixed-size chunking, remember the fact that these chunks may not make sense on their own, and if you pass them to the LLM, chances are it is also confused. Many times there is no relationship between paragraphs and sentences, no context is shared/passed between paragraphs, and it will feel very broken.

The only way to improve the fixed-size chunking is by using the right overlap. Again, using the right overlap is crucial. Having too small an overlap doesn't do any good; having too large an overlap can waste tokens - remember, the context size is limited, and you don't want too much repeated stuff in there. It basically takes up memory in the form of tokens and ultimately causes slower searches.

As a general guideline, a 10 to 15% overlap is usually helpful. If you come across RAG systems with fixed-size chunking overlap at 50% to 70%, stay away from that team. 😏

To implement fixed-size chunking, you can use `CharacterTextSplitter` from LangChain. You then set the `chunk_size` and `chunk_overlap`. If you look into the previous post on "What is RAG," we have a sample code that uses fixed-size chunking.

```python
text = "this is a sample text...."
text_splitter = CharacterTextSplitter(chunk_size=200, chunk_overlap=0)
chunks = text_splitter.split_text(text)
documents = [Document(page_content=chunk) for chunk in chunks]
```

### 2. Sliding Window Chunking

We talked about overlap in the previous section. The sliding window works pretty similar to the overlap, except the overlap basically slides from chunk to chunk while preserving the semantics to a certain extent. It is a better choice of chunking over fixed size but is not the best. You will know why as you complete reading the rest of the chunking strategies in detail.

Let's say your chunk size is 100 tokens, and the stride is 50 tokens. Then your chunks will look like this:

```
Chunk 1:  0 to  99 tokens
Chunk 2: 50 to  99 tokens
Chunk 3: 50 to 100 tokens
```

You might be wondering if the stride is the same as overlap. They are not! Overlap is how much of the previous chunk is reused in the next one, whereas stride is how far the window moves forward after each chunk is created.

```
overlap = chunk size - stride
```

Stride example:
```
chunk size: 100 tokens
stride: 80 tokens
you move the window 80 tokens forward each time
```

Sliding window chunking is a good option to choose if you have documents that are continuous in nature (blogs, articles, etc.), but not an ideal choice for something that is very short - like FAQs.

In the previous fixed chunking, we used `CharacterTextSplitter`. For Sliding Window Chunking, we will use `RecursiveCharacterTextSplitter` from LangChain.

```python
from langchain_text_splitters import RecursiveCharacterTextSplitter

text = (
    "This is a sample document that we are going to split using a sliding window technique. "
    "The goal is to show how overlapping chunks can preserve context from one chunk to another. "
    "This is particularly useful when working with large language models that have limited context windows."
)

# Configure chunking: Sliding Window = chunk_size - overlap
chunk_size = 100
chunk_overlap = 30  # Controls how much the window "slides" (smaller = more overlap)

splitter = RecursiveCharacterTextSplitter(
    chunk_size=chunk_size,
    chunk_overlap=chunk_overlap,
    separators=[" ", ""],  # simple split on whitespace
)

chunks = splitter.split_text(text)

for i, chunk in enumerate(chunks):
    print(f"\n--- Chunk {i+1} ---\n{chunk}")

```

### 3. Recursive chunking
Recursive chunking is an advanced strategy that tries to preserve semantic boundaries (like paragraphs, sentences, or even sections) while still respecting a chunk size limit. It works by recursively breaking your text using a list of delimiters, starting with the most coarse (like paragraphs), and going finer if needed (token/word level).

Recursive Chunking looks exactly like Sliding Window Chunking, uses the same `RecursiveCharacterTextSplitter` splitter module, and the only difference is how it seperates/splits the text. Pay special attention the `separators` parameter to the `RecursiveCharacterTextSplitter` method.

```python
from langchain_text_splitters import RecursiveCharacterTextSplitter

doc = """
## Introduction

This is a guide to recursive chunking. It's awesome.

## Why?

Because cutting a sentence in half is rude, obviously.

## Here's what happens if it's too long...

We try splitting paragraphs. Then sentences. Then words. Then we panic and just slice it!
"""

text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=100,
    chunk_overlap=20,
    separators=["\n\n", "\n", ".", " ", ""],  # ordered from coarse to fine
)

chunks = text_splitter.split_text(doc)

for i, chunk in enumerate(chunks):
    print(f"\n--- Chunk {i+1} ---\n{chunk}")

```

Note that the seperator in the code above uses newline, period and space as delimiters? Please note that the order in which it splits the text matters a lot. So make sure the seperators are defined in the correct order.

In this example, the Recursive Chunking is done by splitting text on paragraphs ("\n\n" and "\n"), and if the lines are long, it seperates using "." (period). If each sentence is too long, it will further seperate using " " (space). if everything fails, it defaults to brute force split based on character length.

It supposedly is better at maintaining semantics as it doesn't randomly cut off sentence in the middle, and it has better readable chunks, which makes retrieval results more coherent and easier for the LLM to work with. 

### 4. Semantic Chunking using AI Models
We've gone through fixed-size chunking, sliding window, and recursive chunking strategies so far. these strategies follow simple rules for splitting the text, and they do not really try to understand the document before making the document into smaller chunks. Semantic chunking, on the other hand looks at the document, understands the meaning of text and then tries to create more meaningful chunks. As you can tell, this takes a bit of effort, and can be slower compared to previous strategies. The up side is that the results will be more accurate and you get quality responses.

Unlike the text splitter classes, the semantic chunking requires use of AI model, and `SentenceTransformer` library helps achieve this functionality.

Let us discuss a scenario where the document we want to chunk has various topics. If we were to use fixed chunking, or sliding window chunking options, the chunks may not make any sense after the splitting is done. The end chunks may have overlapping infromation and if the chunks doens't make sense to us, LLM may not be able to return better responses either. 

```
pip install sentence-transformers
```

```python
from sentence_transformers import SentenceTransformer, util

text = """
Artificial Intelligence is no longer just a buzzword. From virtual assistants that can schedule your meetings to recommendation systems that know you better than your friends, AI is becoming the brain behind modern tech.

Smart home technology is turning homes into responsive, intelligent spaces. From smart thermostats and lights to voice-controlled appliances, convenience is now automated. Your coffee machine can start brewing the moment your alarm goes off.

3D printing is revolutionizing how we build things. Whether you're prototyping a product or printing a spare part for your bike, desktop 3D printers are putting manufacturing power into the hands of everyday users.

Drones have moved far beyond hobbyist toys. They're now used for aerial photography, precision agriculture, and even search-and-rescue missions. With improved battery life and AI-assisted navigation, they're becoming indispensable tools.

A home lab setup used to be the realm of hardcore sysadmins and tinkerers, but not anymore. With affordable rack servers, virtualization tools, and a touch of curiosity, anyone can build their own mini data center. Whether it’s for learning, testing, or just flexing your geek cred, home labs are the ultimate playground.
"""

sentences = [s.strip() for s in text.split('\n') if s.strip()]
model = SentenceTransformer("all-MiniLM-L6-v2")

embeddings = model.encode(sentences, convert_to_tensor=True)

threshold = 0.4
chunks = []
current_chunk = [sentences[0]]

for i in range(1, len(sentences)):
    similarity = util.cos_sim(embeddings[i - 1], embeddings[i]).item()
    if similarity > threshold:
        current_chunk.append(sentences[i])
    else:
        chunks.append(" ".join(current_chunk))
        current_chunk = [sentences[i]]

if current_chunk:
    chunks.append(" ".join(current_chunk))

for i, chunk in enumerate(chunks):
    print(f"\n Chunk {i + 1}:\n{chunk}")

```

This code splits the document into sentenses, embeds each sentense and measures semantic similarity between sentenses, and merges similar ones together to form meaningful chunks. After running the code above, you will notice that all the lines that are "related" are grouped together.
You will see all AI related, Smart Home rlated, 3D printing related, Drone related and Home Lab sentences together.

Semantic Chunking is a very powerful chunking strategy, but comes with a bit of an overhead - takes some cycles, processing power and slower compared to other options. There are other options you can look into, where you can implement multiple chunking strategies to improve quality of responses.

### 5. Metadata-aware or document-structure-based chunking
Metadata-aware chunking strategy comes in handy when you are given a set of documents that have structure. It could be headings, markdown, paragraphs, bullet points, chapters, table of contents, sections, and more.

Similar to the Semantic Chunking, this metadata-aware chunking also preserves semantic meaning and intent. Here is a sample code:

```python
import markdown2
from bs4 import BeautifulSoup
from typing import List, Dict

def parse_markdown_chunks(md_text: str) -> List[Dict]:
    # Convert Markdown to HTML
    html = markdown2.markdown(md_text)

    # Parse HTML and navigate by structure
    soup = BeautifulSoup(html, 'html.parser')
    chunks = []
    current_chunk = ""
    current_heading = "Untitled Section"

    for element in soup.body.children:
        if element.name and element.name.startswith('h'):
            if current_chunk:
                chunks.append({
                    "section": current_heading,
                    "content": current_chunk.strip()
                })
            current_heading = element.get_text()
            current_chunk = ""
        else:
            current_chunk += element.get_text() + "\n"

    if current_chunk:
        chunks.append({
            "section": current_heading,
            "content": current_chunk.strip()
        })

    return chunks

# Sample markdown text
markdown_text = """
# AI Shenanigans

Welcome to the world of AI-powered nonsense. This blog explores how machines are getting snarky and helpful at the same time.

## Smart Home Tech

Your fridge now knows when you run out of cheese. This is either very helpful or very unsettling.

## 3D Printing Awesomeness

From printing parts to printing art, your imagination is the only limit.

## Drones in the Backyard

Delivery? Surveillance? Hobby? Drones have become flying Swiss army knives.

## Home Lab Chronicles

Where servers hum and dreams compile. Every geek needs one.
"""

# Run the chunker
chunks = parse_markdown_chunks(markdown_text)

# Print the structured output
for index, chunk in enumerate(chunks):
    print(f"\nChunk {index+1} - [{chunk['section']}]:\n{chunk['content']}")
```

The above code uses `BeautifulSoup` to parse a markdown text, understands the structure and based on the structure of the document, it splits the document into chunks.

The output will look something like:
```
Chunk 1 - [AI Shenanigans]:
Welcome to the world of AI-powered nonsense. This blog explores how machines are getting snarky and helpful at the same time.

Chunk 2 - [Smart Home Tech]:
Your fridge now knows when you run out of cheese. This is either very helpful or very unsettling.

...
```
If you notice, this is not using `SentenceTransformer` or AI model to get more meaningful chunks. You can certainly combine this with `SentenceTransformer` to make it even better.


### 6. Hybrid Strategy

As I mentioned, there is no silver bullet and there is no one size fits all. It is all about how much understanding you have about the documents that you are trying to ingest, and based on the content of the documents, you should decide which chuking strategy works best for you. You can use one of the options above, or even use combination of multiple chunking strategies in order to get the quality responses you are looking for.

Hybrid Strategies are often more context aware, offer higher quality retrieval and generation. It helps with avoiding "bad chunks", and reduce noise. However, these are more complex to implement, takes more processing power, often slower in performance, and takes longer to deliver.

I apologize for the long post. What started as a quick note turned into a recursive chunking session of its own. One layer led to another, and suddenly I felt like peeling an onion, it kept revealing more layers. Hopefully, it didn’t bring tears to your eyes by the end like it did to me :)

Peace!