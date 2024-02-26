## Visualization: Using displaCy

```python
>>> about_interest_doc = nlp(about_interest_text)
>>> displacy.serve(about_interest_doc, style="dep")

In [1]: displacy.render(about_interest_doc, style="dep", jupyter=True)
```

The above code will spin up a simple web server. You can then see the visualization by going to http://127.0.0.1:5000 in your browser:

![Vialualization](https://robocrop.realpython.net/?url=https%3A//files.realpython.com/media/displacy_pos_tags.45059f2bf851.png&w=876&sig=901628ce2a8ce856de3715d9f253aabce2da1c48)

NLP-Sideway/media/spaCy/download.webp


```python 
def is_token_allowed(token):
...     return bool(
...         token
...         and str(token).strip()
...         and not token.is_stop
...         and not token.is_punct
...     )
...
>>> def preprocess_token(token):
...     return token.lemma_.strip().lower()
...
>>> complete_filtered_tokens = [
...     preprocess_token(token)
...     for token in complete_doc
...     if is_token_allowed(token)
... ]
```

complete_filtered_tokens doesn’t contain any stop words or punctuation symbols, and it consists purely of lemmatized lowercase tokens

## Rule-Based Matching Using spaCy

Rule-based matching is one of the steps in extracting information from unstructured text. It’s used to identify and extract tokens and phrases according to patterns (such as lowercase) and grammatical features (such as part of speech).

While you can use regular expressions to extract entities (such as phone numbers), rule-based matching in spaCy is more powerful than regex alone, because you can include semantic or grammatical filters.

For example, with rule-based matching, you can extract a first name and a last name, which are always proper nouns:

```python
>>> from spacy.matcher import Matcher
>>> matcher = Matcher(nlp.vocab)

>>> def extract_full_name(nlp_doc):
...     pattern = [{"POS": "PROPN"}, {"POS": "PROPN"}]
...     matcher.add("FULL_NAME", [pattern])
...     matches = matcher(nlp_doc)
...     for _, start, end in matches:
...         span = nlp_doc[start:end]
...         yield span.text
...

>>> next(extract_full_name(about_doc))
'Gus Proto'


###################################################
 def extract_phone_number(nlp_doc):
...     pattern = [
...         {"ORTH": "("},
...         {"SHAPE": "ddd"},
...         {"ORTH": ")"},
...         {"SHAPE": "ddd"},
...         {"ORTH": "-", "OP": "?"},
...         {"SHAPE": "dddd"},
...     ]
...     matcher.add("PHONE_NUMBER", None, pattern)
...     matches = matcher(nlp_doc)
...     for match_id, start, end in matches:
...         span = nlp_doc[start:end]
...         return span.text
...

>>> conference_org_doc = nlp(conference_org_text)
>>> extract_phone_number(conference_org_doc)
'(123) 456-7891'
```

*ORTH* matches the exact text of the token.
*SHAPE* transforms the token string to show orthographic features, d standing for digit.
*OP* defines operators. Using ? as a value means that the pattern is optional, meaning it can match 0 or 1 times.

## Dependency Parsing Using spaCy

Dependency parsing is the process of extracting the dependency graph of a sentence to represent its grammatical structure. It defines the dependency relationship between headwords and their dependents. The head of a sentence has no dependency and is called the root of the sentence. The verb is usually the root of the sentence. All other words are linked to the headword.

The dependencies can be mapped in a directed graph representation where:

Words are the nodes.
Grammatical relationships are the edges.
Dependency parsing helps you know what role a word plays in the text and how different words relate to each other

