---
toc: true
layout: post
description: Introduction to NLP for QA
categories: [markdown]
title: NLP for Automated Question Answering
---
# NLP for Automated Question Answering

Welcome to the first edition of the Cloudera Fast Forward blog on Natural Language Processing for Question Answering! Throughout this series, we’ll build a Question Answering (QA) system with off-the-shelf parts and blog about our process and what we find along the way. We hope to wind up with a beginning-to-end documentary that provides:

- insight into QA as a tool, 
- useful context to make decisions for those who might build their own QA
 system,
- tips and tricks we pick up as we go,
- sample code, if we’re lucky (we’ll show our work in notebooks), and
- entertainment, as you watch us fumble our way through the process. (Feedback
 and commentary welcome.)

We’re trying a new thing here. In the past, Cloudera Fast Forward has
 documented its work in [discrete reports](https://www.cloudera.com/products/fast-forward-labs-research/fast-forward-labs-research-reports.html). 
 We hope this new format suits the above goals and makes the topic more
  accessible, while ultimately being useful. 

To kick off the series, this introductory post will discuss what QA is and isn’t, where this technology is being employed, and what techniques are used to accomplish this natural language task. 

## Question Answering in a Nutshell
Question Answering is a human-machine interaction to extract information from 
data using natural language queries. Machines do not inherently understand human
 languages any more than the average human understands machine language. A 
 well-developed QA system bridges the gap between the two, allowing humans to 
 extract knowledge from data in a way that is natural to us, i.e., asking questions. 

QA systems accept questions in the form of natural language (typically text 
based, although you are probably also familiar with systems that accept speech 
input, such as Amazon’s Alexa or Apple’s Siri), and output a concise answer. 
Google’s search engine product adds a form of question answering in addition to 
its traditional search results, as illustrated here: 

![]({{ site.baseurl }}/images/post1/abe_search_crop.png)

Google took our question and returned a set of 1.3 million documents (not shown)
 relevant to the search terms, i.e., documents about Abraham Lincoln. 
 Google also used what it knows about the contents of some of those documents 
 to provide a “[snippet](https://support.google.com/websearch/answer/9351707?p=featured_snippets)” 
 that answered our question in one word, presented above 
 a link to the most pertinent website and keyword-highlighted text. 

This goes beyond the standard capabilities of a search engine, which typically only return a list of relevant documents or websites. Google is pretty tight-lipped about its algorithms, so we can’t tell you whether they are harnessing elements of natural language understanding or which algorithms they’re using. We’ll revisit this example in a later section and discuss how we can (and will!) build our own system to demonstrate how question answering works in practice. 


## Why Question Answering? 
Sophisticated Google searches with precise answers are fun, but how useful are 
QA systems in general? It turns out that this technology is maturing rapidly. 
Gartner recently identified [natural language processing and conversational 
analytics](https://www.gartner.com/smarterwithgartner/gartner-top-10-data-analytics-trends/) 
as one of the top trends poised to make a substantial impact in the 
next three to five years.  These technologies will provide increased data access, 
ease of use, and wider adoption of analytics platforms - especially to 
mainstream users. QA systems specifically will be a core part of the NLP suite, 
and are already seeing adoption in several areas.


## Designing a Question Answerer 
As explained above, question answering systems process natural language queries and output concise answers. This general capability can be implemented in dozens of ways. How a QA system is designed depends, in large part, on three key elements: the knowledge provided to the system, the types of questions it can answer, and the structure of the data supporting the system.  

### Domain
QA systems operate within a domain, constrained by the data that is provided to 
them. The domain represents the embodiment of all the knowledge the system can 
know. There are two domain paradigms: open and closed. Closed domain systems 
are narrow in scope and focus on a specific topic or regime. Open domain systems 
are broad, answering general knowledge questions. 


### Implementation 
There’s more than one way to cuddle a cat, as the saying goes. Question 
answering seeks to extract information from data and, generally speaking, data 
come in two broad formats: structured and unstructured. QA algorithms have been 
developed to harness the information from either paradigm: knowledge- based 
systems for structured data and information retrieval-based systems for 
unstructured (text) data. Some QA systems exploit a hybrid design that harvests 
information from both data types; IBM’s Watson is a famous example. In this 
section, we’ll highlight some of the most widely used techniques in each data 
regime - concentrating more on those for unstructured data, since this will be 
the focus of our applied research. Because we’ll be discussing explicit methods 
and techniques, the following sections are more technical. And we’ll note that, 
while we provide an overview here, an even more comprehensive discussion can be 
found in the Question Answering chapter of Jurafsky and Martin’s Speech and 
Language Processing textbook. 


![]({{ site.baseurl }}/images/post1/reading_retriever.jpg "Get it? Retriever? Reader?")

#### Information Retrieval-Based Systems: Retrievers and Readers

Information retrieval-based question answering (IR QA) systems find and extract 
a text segment from a large collection of documents. The collection can be as 
vast as the entire web (open domain) or as specific as a company’s Confluence 
documents (closed domain). Contemporary IR QA systems first identify the most 
relevant documents in the collection, and then extract the answer from the 
contents of those documents. To illustrate this approach, let’s revisit our 
Google example from the introduction, only this time we’ll include some of the 
search results!

![]({{ site.baseurl }}/images/post1/abe_search.png "Did Abe have big
 ears?")


We already talked about how the snippet box acts like a QA system. 
The search results below the snippet illustrate some of the reasons why an IR QA 
system can be more useful than a search engine alone. The relevant links vary 
from essentially advertising (study.com), to making fun of Lincoln’s ears 
(Reddit at its finest), to a discussion of color blindness (answers.com without 
the answer we want), to an article about all presidents’ eye color (getting 
warmer, Chicago Tribune), to the last link, answers.yahoo.com, which is on-topic, 
and narrowly scoped to Lincoln but gives an ambiguous answer. Without the 
snippet box at the top, a user would have to skim each of these links looking 
for their answer. 

IR QA systems are not just search engines, which take general natural language 
terms and provide a list of relevant documents. IR QA systems perform an 
additional layer of processing on the most relevant documents to deliver a 
pointed answer based on the contents of those documents (like the snippet box). 
While we won’t hazard a guess at exactly how Google extracted “gray” from these 
search results, we can examine how an IR QA system could exhibit similar 
functionality in a real world (e.g., non-Google) implementation. Below we 
illustrate the workflow of a generic IR-based QA system. These systems generally 
have two main components: the document retriever and the document reader.  

![]({{ site.baseurl }}/images/post1/QAworkflow.png "Generic IR QA
 system")


The document retriever functions as the search engine, ranking and retrieving 
relevant documents to which it has access. It supplies a set of candidate 
documents that could answer the question (often with mixed results, per the 
Google search shown above). The second component is the document reader: 
reading comprehension algorithms built with core NLP techniques. This component 
processes the candidate documents and extracts from one of them an explicit span 
of text that best satisfies the query. Let’s dive into each of these components. 