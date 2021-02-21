---
title: "Adding Lemmatization to CountVectorizer Without Making a Mess"
date: 2021-02-21T12:57:33-05:00
slug: "" 
description: "We write a custom tokenizer for scikit-learns CountVectorizer that implements lemmatization with parts of speech tagging."
keywords: ["scikit-learn", "NLTK", "Lemmatization", "CountVectorizer", "Natural Language Processing", "NLP", "Data Science", "Data Cleaning"]
draft: false
tags: ["DataJobs", "Data Cleaning", "NLP"]
math: false
toc: false
---

## Background

This past week, I have been working on refactoring some of the code for my [DataJobs](https://github.com/sethchart/DataJobs) project. 
That project explores the relationship between job descriptions and job titles in the data industry using natural language processing techniques.
In particular, the DataJobs project uses Latent Dirichlet Allocation to detect topics within job descriptions. 
In this post, I am going to focus on data preprocessing for LDA and how to implement it in an scikit-learn pipeline.

## The Main Problem

For my DataJobs project, I wanted to process documents and convert them to word count vectors, which can then be used as input for LDA.
[Scikit-learn](https://scikit-learn.org/stable/index.html) provides a useful and well documented tool for word count vectorization called [CountVectorizer](https://scikit-learn.org/stable/modules/generated/sklearn.feature_extraction.text.CountVectorizer.html).
CountVectorizer covered almost all of my preprocessing needs by default, except for lemmatization.

Lemmatization is the process of replacing each word in a document with the root word that appears at the top of its dictionary definition.
Linguists call this root word the _lemma_. 
For example, the lemma of the word 'taught' is 'teach'.

When I read through the [feature extraction user guide](https://scikit-learn.org/stable/modules/feature_extraction.html#text-feature-extraction), I found the following is section 6.2.3.10.

>Fancy token-level analysis such as stemming, lemmatizing, compound splitting, filtering based on part-of-speech, etc. are not included in the scikit-learn codebase, but can be added by customizing either the tokenizer or the analyzer. Here’s a CountVectorizer with a tokenizer and lemmatizer using NLTK:

```python
from nltk import word_tokenize          
from nltk.stem import WordNetLemmatizer 
class LemmaTokenizer:
    def __init__(self):
        self.wnl = WordNetLemmatizer()
    def __call__(self, doc):
        return [self.wnl.lemmatize(t) for t in word_tokenize(doc)]

vect = CountVectorizer(tokenizer=LemmaTokenizer()) 
``` 
>(Note that this will not filter out punctuation.)

The issue here is that I can easily add lemmatization to my CountVectorizer, but if I do, I must also provide my own text cleaning to remove punctuation, lowercase text, and clean-up special characters.

## My Solution

Using the code snippet above as a starting point, I decided to build my own tokenizer to implement text cleaning and lemmatization. 

``` python
class LemmaTokenizer:
    """LemmaTokenizer. Provides text cleaning and lemmatization as a tokenizer
    for use with CountVectorizer.
    """

    def __init__(self):
        """__init__.
        """
        self.lemmatizer = WordNetLemmatizer()

    def __call__(self, doc):
        """__call__.

        Parameters
        ----------
        doc :
            doc is a document encoded as a string.
        """
        return self.doc_lemmatizer(doc)

    def doc_lemmatizer(self, doc: str) -> List[str]:
        """doc_lemmatizer. Lemmatize tagged words from the provided doc.

        Parameters
        ----------
        doc : str
            doc is a document encoded as a string.

        Returns
        -------
        List[str]

        """
        doc_tags = self.doc_tagger(doc)
        lemmatized_doc = [
            lemma.translate(str.maketrans('', '', string.punctuation)) for
            sentence_tags in doc_tags for
            lemma in self.sentence_lemmatizer(sentence_tags) if
            lemma.translate(str.maketrans('', '', string.punctuation)) != ''
        ]
        return lemmatized_doc

    def doc_tagger(self, doc: str) -> List[List[POS_Tag]]:
        """doc_tagger. Takes a document and returns parts of speech tagged
        tokens organized as a list of sentences each of which is a list of POS
        tagged words.

        Parameters
        ----------
        doc : str
            doc is a document encoded as a string.

        Returns
        -------
        List[List[POS_Tag]]

        """
        pos_tags = [
            self.sentence_tagger(sentence) for
            sentence in self.doc_tokenizer(doc)
        ]
        return pos_tags

    def sentence_tagger(self, sentence: List[str]) -> List[POS_Tag]:
        """sentence_tagger. Takes a sentence as a list of tokens and
        returns a list of wordnet POS tagged tokens.

        Parameters
        ----------
        sentence : List[str]
            sentence is a list of word tokens from a sentence.

        Returns
        -------
        List[POS_Tag]

        """
        treebank_tags = pos_tag(sentence)
        wordnet_tags = [
            (treebank_tag[0], self.get_wordnet_pos(treebank_tag[1]))
            for treebank_tag in treebank_tags
        ]
        return wordnet_tags

    def doc_tokenizer(self, doc: str) -> List[List[str]]:
        """doc_tokenizer. Reads in a raw text document and returns a list of
        sentences each represented as a list of words. Text is converted to
        lowercase and newline characters are removed.

        Parameters
        ----------
        doc : str
            doc is a document encoded as a string.
        Returns
        -------
        List[List[str]]

        """
        sentences = sent_tokenize(
            doc.lower().replace('\\n', '')
        )
        doc_tokens = [word_tokenize(sentence) for sentence in sentences]
        return doc_tokens

    def sentence_lemmatizer(self, sentence_tags: List[POS_Tag]) -> List[str]:
        """sentence_lemmatizer. Takes a POS tagged sentence and returns a list
        of word lemmas.

        Parameters
        ----------
        sentence_tags : List[POS_Tag]
            sentence_tags is a list of POS tagged tokens that for a sentence.

        Returns
        -------
        List[str]

        """
        lemmatized_sentence = [
            self.lemmatize(pos_tag) for pos_tag in sentence_tags
        ]
        return lemmatized_sentence

    @staticmethod
    def get_wordnet_pos(treebank_tag: POS_Tag) -> str:
        """get_wordnet_pos. Converts a treebank POS tag to a wordnet POS tag.

        Parameters
        ----------
        treebank_tag : POS_Tag
            treebank_tag first position is the token, second position is the
            treebank POS tag.

        Returns
        -------
        str
        """
        if treebank_tag.startswith('J'):
            tag = wordnet.ADJ
        elif treebank_tag.startswith('V'):
            tag = wordnet.VERB
        elif treebank_tag.startswith('N'):
            tag = wordnet.NOUN
        elif treebank_tag.startswith('R'):
            tag = wordnet.ADV
        else:
            tag = ''
        return tag

    def lemmatize(self, pos_tag: POS_Tag) -> str:
        """lemmatize. Takes a POS tagged word and returns its lemma.

        Parameters
        ----------
        pos_tag : POS_Tag
            pos_tag first position is a word token, second position is a
            wordnet POS tag.

        Returns
        -------
        str
        """
        if pos_tag[1] != '':
            lemmatized_word = self.lemmatizer.lemmatize(pos_tag[0], pos_tag[1])
        else:
            lemmatized_word = pos_tag[0]
        return lemmatized_word
```

## Conclusion

This tokenizer implements two major improvements on the example class from the scikit-learn user guide.
1. Punctuation is dropped in the doc_lemmatizer function. This happens after lemmatization so that we can take advantage of punctuation to detect sentence structure.
2. We use parts of speech tagging to make the lemmatizer more aware of context. For example the word "meeting" can appear in a sentence as a noun or as a verb. The word "meeting" is the lemma when "meeting" is a noun, but "meet" is the lemma when "meeting" is a verb. By default WordNetLemmatizer treats every word as a noun. 
