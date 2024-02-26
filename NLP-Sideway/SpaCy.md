[Source](https://realpython.com/natural-language-processing-spacy-python/)

# Introduction to NLP and spaCy

NLP is a subfield of artificial intelligence, and it’s all about allowing computers to comprehend human language. NLP involves analyzing, quantifying, understanding, and deriving meaning from natural languages.


## Installation

```python 
(venv) $ python -m spacy download en_core_web_sm
```

```python
>>> import spacy
>>> nlp = spacy.load("en_core_web_sm")
>>> nlp
<spacy.lang.en.English at 0x291003a6bf0>
```
The load() function returns a Language callable object, which is commonly assigned to a variable called nlp.

## The Doc Object

```python

>>> introduction_doc = nlp(
...     "This tutorial is about Natural Language Processing in spaCy."
... )


>>> import pathlib
>>> file_name = "introduction.txt"
>>> introduction_doc = nlp(pathlib.Path(file_name).read_text(encoding="utf-8"))
>>> type(introduction_doc)
spacy.tokens.doc.Doc
>>> print ([token.text for token in introduction_doc])
['This', 'tutorial', 'is', 'about', 'Natural', 'Language',
'Processing', 'in', 'spaCy', '.', '\n']
```

## Sentence Detection

Sentence detection is the process of locating where sentences start and end in a given text. This allows you to you divide a text into linguistically meaningful units. You’ll use these units when you’re processing your text to perform tasks such as part-of-speech (POS) tagging and named-entity recognition

```python
>>> about_doc = nlp(about_text)
>>> sentences = list(about_doc.sents)
>>> len(sentences)
```

In the above example, spaCy is correctly able to identify the input’s sentences. With .sents, you get a list of Span objects representing individual sentences. You can also slice the [Span](https://spacy.io/api/span) objects to produce sections of a sentence

You can also customize sentence detection behavior by using custom delimiters. Here’s an example where an ellipsis (...) is used as a delimiter, in addition to the full stop, or period (.):

```python
>>> ellipsis_text = (
...     "Gus, can you, ... never mind, I forgot"
...     " what I was saying. So, do you think"
...     " we should ..."
... )

>>> from spacy.language import Language
>>> @Language.component("set_custom_boundaries")
... def set_custom_boundaries(doc):
...     """Add support to use `...` as a delimiter for sentence detection"""
...     for token in doc[:-1]:
...         if token.text == "...":
...             doc[token.i + 1].is_sent_start = True
...     return doc
...

>>> custom_nlp = spacy.load("en_core_web_sm")
>>> custom_nlp.add_pipe("set_custom_boundaries", before="parser")
>>> custom_ellipsis_doc = custom_nlp(ellipsis_text)
>>> custom_ellipsis_sentences = list(custom_ellipsis_doc.sents)
>>> for sentence in custom_ellipsis_sentences:
...     print(sentence)
...
Gus, can you, ...
never mind, I forgot what I was saying.
So, do you think we should ...
```

The @Language.component("set_custom_boundaries") decorator to define a new function that takes a Doc object as an argument. The job of this function is to identify tokens in Doc that are the beginning of sentences and mark their .is_sent_start attribute to True. Once done, the function must return the Doc object again.

Then, you can add the custom boundary function to the Language object by using the .add_pipe() method. Parsing text with this modified Language object will now treat the word after an ellipse as the start of a new sentence

```python
>>> for token in about_doc:
...     print (token, token.idx)
...
Gus 0
Proto 4
is 10
a 13
Python 15
developer 22
```

```python
>>> print(
...     f"{"Text with Whitespace":22}"
...     f"{"Is Alphanumeric?":15}"
...     f"{"Is Punctuation?":18}"
...     f"{"Is Stop Word?"}"
... )
>>> for token in about_doc:
...     print(
...         f"{str(token.text_with_ws):22}"
...         f"{str(token.is_alpha):15}"
...         f"{str(token.is_punct):18}"
...         f"{str(token.is_stop)}"
...     )
...
Text with Whitespace  Is Alphanum?   Is Punctuation?   Is Stop Word?
Gus                   True           False             False
Proto                 True           False             False
is                    True           False             True
a                     True           False             True
Python                True           False             False
```
*.text_with_ws* prints the token text along with any trailing space, if present.
*.is_alpha*+ indicates whether the token consists of alphabetic characters or not.
*.is_punct* indicates whether the token is a punctuation symbol or not.
*.is_stop* indicates whether the token is a stop word or not. You’ll be covering stop words a bit later in this tutorial.

To include the @ symbol as a custom infix, you need to build your own Tokenizer object:

```python
>>> import re
>>> from spacy.tokenizer import Tokenizer

>>> custom_nlp = spacy.load("en_core_web_sm")
>>> prefix_re = spacy.util.compile_prefix_regex(
...     custom_nlp.Defaults.prefixes
... )
>>> suffix_re = spacy.util.compile_suffix_regex(
...     custom_nlp.Defaults.suffixes
... )

>>> custom_infixes = [r"@"]

>>> infix_re = spacy.util.compile_infix_regex(
...     list(custom_nlp.Defaults.infixes) + custom_infixes
... )

>>> custom_nlp.tokenizer = Tokenizer(
...     nlp.vocab,
...     prefix_search=prefix_re.search,
...     suffix_search=suffix_re.search,
...     infix_finditer=infix_re.finditer,
...     token_match=None,
... )

>>> custom_tokenizer_about_doc = custom_nlp(custom_about_text)

>>> print([token.text for token in custom_tokenizer_about_doc[8:15]])
['for', 'a', 'London', '@', 'based', 'Fintech', 'company']
```
- *Vocab*: A storage container for special cases, which is used to handle cases like contractions and emoticons.
- *prefix_search*: A function that handles preceding punctuation, such as opening parentheses.
- *suffix_search*: A function that handles succeeding punctuation, such as closing parentheses.
- *infix_finditer*: A function that handles non-whitespace separators, such as hyphens.
- *token_match*: An optional Boolean function that matches strings that should never be split. It overrides the previous rules and is useful for entities like URLs or numbers.

When you call the Tokenizer constructor, you pass the .search() method on the prefix and suffix regex objects, and the .finditer() function on the infix regex object. Now you can replace the tokenizer on the custom_nlp object

## Stop Words

Stop words are typically defined as the most common words in a language. In the English language, some examples of stop words are the, are, but, and they. Most sentences need to contain stop words in order to be full sentences that make grammatical sense.

With NLP, stop words are generally removed because they aren’t significant, and they heavily distort any word frequency analysis. spaCy stores a list of stop words for the English language:

```python
>>> import spacy
>>> spacy_stopwords = spacy.lang.en.stop_words.STOP_WORDS
>>> len(spacy_stopwords)
326
>>> for stop_word in list(spacy_stopwords)[:10]:
...     print(stop_word)
...
using
becomes
had
itself
once
often
is
herein
who
too

>>> print([token for token in about_doc if not token.is_stop])
```

## Lemmatization

Lemmatization is the process of reducing inflected forms of a word while still ensuring that the reduced form belongs to the language. This reduced form, or root word, is called a lemma.

For example, organizes, organized and organizing are all forms of organize. Here, organize is the lemma. The inflection of a word allows you to express different grammatical categories, like tense (organized vs organize), number (trains vs train), and so on. Lemmatization is necessary because it helps you reduce the inflected forms of a word so that they can be analyzed as a single item. It can also help you normalize the text.

```python
>>> for token in conference_help_doc:
...     if str(token) != str(token.lemma_):
...         print(f"{str(token):>20} : {str(token.lemma_)}")
...
                  is : be
                  He : he
               keeps : keep
          organizing : organize
             meetups : meetup
               talks : talk
```
## Word Frequency

```python
from collections import Counter

>>> words = [
...     token.text
...     for token in complete_doc
...     if not token.is_stop and not token.is_punct
... ]

>>> print(Counter(words).most_common(5))
[('Gus', 4), ('London', 3), ('Natural', 3), ('Language', 3), ('Processing', 3)]
```

Four out of five of the most common words are stop words that don’t really tell you much about the summarized text. This is why stop words are often considered noise for many applications.

## Part of speech
Part of speech or POS is a grammatical role that explains how a particular word is used in a sentence. There are typically eight parts of speech:

Noun
Pronoun
Adjective
Verb
Adverb
Preposition
Conjunction
Interjection
Part-of-speech tagging is the process of assigning a POS tag to each token depending on its usage in the sentence. POS tags are useful for assigning a syntactic category like noun or verb to each word.

```python
>>> about_doc = nlp(about_text)
>>> for token in about_doc:
...     print(
...         f"""
... TOKEN: {str(token)}
... =====
... TAG: {str(token.tag_):10} POS: {token.pos_}
... EXPLANATION: {spacy.explain(token.tag_)}"""
...     )
...
TOKEN: Gus
=====
TAG: NNP        POS: PROPN
EXPLANATION: noun, proper singular

TOKEN: Proto
=====
TAG: NNP        POS: PROPN
EXPLANATION: noun, proper singular

TOKEN: is
=====
TAG: VBZ        POS: AUX
EXPLANATION: verb, 3rd person singular present
```
*.tag_* displays a fine-grained tag.
*.pos_* displays a coarse-grained tag, which is a reduced version of the fine-grained tags

*spacy.explain()* to give descriptive details about a particular POS tag, which can be a valuable reference tool.

