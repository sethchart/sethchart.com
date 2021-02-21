---
title: "Latent Dirichlet Allocation"
date: 2021-02-14T13:24:44-05:00
slug: ""
description: ""
keywords: [LDA, NLP, Machine Learning]
draft: false
tags: []
math: false
toc: false
---
## An Example

In one of my recent projects I wanted to investigate job postings for jobs in the data industry.
In the data industry, it is widely believed that job titles are not a reliable indicator of the role that they describe. 
The title Data Scientist may describe two completely different roles depending on the company.
Similarly, nearly identical roles might have the job title Data Scientist in one company and Data Analyst in another company. 
I wanted to investigate this belief by using unsupervised learning to 'read' full job descriptions, cluster them into similar roles, and then investigate the relationship between these roles and the stated job titles.
If you would like to learn more about that project please check our the project repo [here](https://github.com/sethchart/DataJobs).

In this post, I want to focus in on the process of 'reading' the job descriptions.
Ideally, an algorithm that reads the job descriptions will take the full text of the job description and return a simple and meaningful summary that can be used to group similar job descriptions together.

## How Would a Human Read?

If was forced to read a large collection of job description and group similar roles together by hand, I would probably read through the job description and try to find all of the distinct skills required by the role and summarize them as a short list. Then, I would group jobs together that required similar skills.
Most likely, each job description would describe several skills with varying emphasis.

## What is Latent Dirichlet Allocation?

Latent Dirichlet Allocation (LDA), is a model that approximates the reading process above quite well.
Essentially, we give LDA a collection of documents and ask for it to detect common topics within the documents.
We must supply LDA with the number of topics to detect. 
Once LDA has been trained on the collection of documents, it summarizes each document by describing the mixture of topics that it describes. 
Even better, a trained model can summarize a new document that it has never seen.
Note that, a new document should be similar to the training data.
An LDA model trained on job descriptions has no business reading takeout menus. 

So how does this apply to the task of summarizing required skills by reading job descriptions?
Well the topics in well written job descriptions should be, for the most part, required skills. Therefore, LDA should detect required skills when it is trained on job descriptions. 

## How Does LDA Work?

In this section I want to give a fairly non-technical overview of LDA ignoring most of the probabilistic details. 

### How does LDA See a Document?

The first thing that we should know about LDA is that it does not know anything about word order.
The input for an LDA model is a [bag-of-words](https://en.wikipedia.org/wiki/Bag-of-words_model), which reduces a document to a word frequency distribution.
For example, consider the document below, consisting of two sentences.

```python
doc1 = "Seth likes to go for walks. Salimah likes walks too."
```
When we convert the document to a bag-of-words, we get the following.
```python
bow1 = { 
    "seth": 1, "likes": 2, "to": 1, "go": 1, 
    "for": 1, "walks": 2, "salimah": 1, "too": 1 
}

```
### What About a Collection of Documents?
Generally we apply LDA to a collection of documents, often called a _corpus_.
Let's add two more documents to our corpus.

```python
doc2 = "Salimah likes to travel."

doc3 = """Salimah likes pasta more than ramen.
Seth likes ramen more than pasta."""
```

The corresponding bag-of-words representations are below.
```python
bow2 = { 
    "salimah": 1, "likes": 1, "to": 1, "travel": 1
}

bow3 = { 
    "salimah": 1, "likes": 2, "pasta": 2, 
    "more": 2, "than": 2, "ramen": 2, "seth": 1
}
```
Now or corpus is the set of all three documents:
```python
corpus = { doc1, doc2, doc3 }
```

The set of all words that appear in any document in our corpus is called our _vocabulary_.
For our example corpus the vocabulary is the set of words below.

```python
vocab = {
   "seth", "likes", "to", "go", "for", "walks",
   "salimah", "too", "travel", "pasta", "more",
   "than", "ramen"
}
```

### A Model for Writing

Let's imagine that when I wrote the documents above I had one of three topics in mind: exercise, food, or hobbies.
When writing the first document I had the topic of exercise in mind, so I selected words that were relevant to that topic to include.
Similarly, the second document is on the topic of food and the third is on the topic of hobbies. 
In this case, each document is essentially focused on a single topic. 

Suppose that I tell you that I am about to write a new document on the topic of food, using the vocabulary from our corpus. 
It seems probable that I will use the words `pasta` and `ramen`, but much less probable that I will use the word `walk`. 
If instead, I told you that the topic of the new document would be exercise, then it seems probable that I will use the word `walk`, but less probable that I will use the word `ramen`.

A document does not need to have a single topic.
To use our examples efficiently, let's imagine that instead of three topics, I had two topics in mind: Seth's preferences and Salimah's preferences. 
Both Document 1 and Document 3 describe a mixture of my two topics, while Document 2 only describes Salimah's preferences.

An LDA model makes the following assumptions about a corpus of documents.

1. There is a fixed vocabulary of words that were used to construct the documents in our corpus.
2. There is a fixed set of topics that are described by the documents in our corpus.
3. For each word in our vocabulary, we can assign a conditional probability that the word will be included in a document given the current topic.
4. For each topic, we can assign a probability that it will be described by a document in our corpus. 

Based on these assumptions, an LDA model assumes that each document is written according to the following process.

1. Select the length of the document at random.
2. Select a distribution of topics for the document according to the corpus wide distribution.
3. For each word in the document, first select a particular topic according to the distribution from Step 2, then select a word from the vocabulary according to the conditional distribution on the vocabulary given the current topic.

## Fitting an LDA Model

Above we have described at length the underling assumptions of an LDA model, particularly the model for writing a document described in the last section. 
The issue that we still need to resolve has to do with what we know when we are handed a corpus of documents.

For each document in our corpus, we can observe the frequency of each word from our vocabulary. 
However, we do not know the distribution of topics for our corpus, the conditional distributions on the vocabulary given the topic, or the distribution of topics described in each document.

We are primarily interested in the distribution of topics that are described in a document. 
However, to access this information we also need to determine the topic distribution and the conditional word distributions.

There are two main computational tasks that must be completed to obtain topic distributions from documents.
1. Estimate the distribution of topics and conditional distributions of words that maximize the log likelihood of the training data.
2. Given the distributions from Step 1, estimate the distribution of topics described by a particular document.

Provided that we have completed Step 1 and a document is reasonably similar to the training data, we can execute Step 2 on a document that was not part of the training data. 

A deeper explanation of the steps above requires a substantially more detailed description of the probabalistic tools that are used and is beyond the scope of this post.
We refer the interested reader to the reference below, which was the inspiration for this post.

## Conclusion

LDA models provide a powerful unsupervised tool for detecting topics within a corpus of documents. 
They support documents that describe a mixture of topics.
The number of topics must be selected by the prectitioner and tuning the number of topics can be a computationaly intensive process. 
Because an LDA model is based on a realistic model of document creation and explicitly incorporates the concept of topics, this type of model can provide more intuitive results than methods that rely on word embedding geoometry.

## References
[David M. Blei, Andrew Y. Ng, and Michael I. Jordan. 2003. Latent dirichlet allocation. J. Mach. Learn. Res. 3, null (3/1/2003), 993–1022.](https://dl.acm.org/doi/pdf/10.5555/944919.944937)
