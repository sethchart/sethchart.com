---
title: "Data Cleaning for Latent Dirichlet Analysis"
date: 2021-02-21T11:39:06-05:00
slug: ""
description: "We write a custom tokenizer for scikit-learn's CountVectorizer that implements lemmatization with parts of speach tagging."
keywords: [NLP, Data Processing, scikit-learn, lemmatization]
draft: false
tags: [NLP, Data Processing, DataJobs]
math: false
toc: false
---


## Background

LDA treats each word in a document as though it was placed there by selecting it at random and independent of all other words in the document.
This is clearly not true for almost every document that we will want to analyze.

### Multiple Word Forms
One major issue with the independence assumption in LDA comes from the different forms of a word, for example 'teach', 'taught', 'teaching', and 'teaches' are all words that can represent the same essential concept in our text.
These words are not really independent of one-another, they are all different variations on the word 'teach'.
Linguists call the root word 'teach' the _lemma_ for all for four words.
The lemma is the word you would look up in the dictionary.
To bring our document closer to the assumptions of LDA we can replace the words in our document with their lemmas.
The process of reducing all words in a document to their lemmas is called lemmatization.
After lemmatization, my document would no longer have four highly dependent words 'teach', 'taught', 'teaching', and 'teaches', instead it would have a single lemma 'teach'.

### Phrases

Phrases are sequences of a few words that are combined in a particular order to convey some particular meaning.
This is another challenge to our assumption of independence in LDA. 
The words 'data' and 'science' will show up in documents as the phrase 'data science' quite often, so these two words are not independent.
To bring our document closer to the assumptions of LDA, we can replace each occurrence
