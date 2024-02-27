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

```python
>>> import spacy
>>> nlp = spacy.load("en_core_web_sm")
>>> piano_text = "Gus is learning piano"
>>> piano_doc = nlp(piano_text)
>>> for token in piano_doc:
...     print(
...         f"""
... TOKEN: {token.text}
... =====
... {token.tag_ = }
... {token.head.text = }
... {token.dep_ = }"""
...     )
...
TOKEN: Gus
=====
token.tag_ = 'NNP'
token.head.text = 'learning'
token.dep_ = 'nsubj'

TOKEN: is
=====
token.tag_ = 'VBZ'
token.head.text = 'learning'
token.dep_ = 'aux'
```

nsubj is the subject of the word, and its headword is a verb.
aux is an auxiliary word, and its headword is a verb.
dobj is the direct object of the verb, and its headword is also a verb.
The list of relationships isn’t particular to spaCy. Rather, it’s an evolving field of linguistics research.

```python
>>> displacy.serve(piano_doc, style="dep")
```
![Image](https://files.realpython.com/media/displacy_dependency_parse.de72f9b1d115.png)

## Tree and Subtree Navigation

The dependency graph has all the properties of a tree. This tree contains information about sentence structure and grammar and can be traversed in different ways to extract relationships.

spaCy provides attributes like .children, .lefts, .rights, and .subtree to make navigating the parse tree easier. Here are a few examples of using those attributes:

```python
>>> import spacy
>>> nlp = spacy.load("en_core_web_sm")
>>> one_line_about_text = (
...     "Gus Proto is a Python developer"
...     " currently working for a London-based Fintech company"
... )
>>> one_line_about_doc = nlp(one_line_about_text)

>>> # Extract children of `developer`
>>> print([token.text for token in one_line_about_doc[5].children])
['a', 'Python', 'working']

>>> # Extract previous neighboring node of `developer`
>>> print (one_line_about_doc[5].nbor(-1))
Python

>>> # Extract next neighboring node of `developer`
>>> print (one_line_about_doc[5].nbor())
currently

>>> # Extract all tokens on the left of `developer`
>>> print([token.text for token in one_line_about_doc[5].lefts])
['a', 'Python']

>>> # Extract tokens on the right of `developer`
>>> print([token.text for token in one_line_about_doc[5].rights])
['working']

>>> # Print subtree of `developer`
>>> print (list(one_line_about_doc[5].subtree))
[a, Python, developer, currently, working, for, a, London, -, based, Fintech
company]
```


## Shallow Parsing
Shallow parsing, or chunking, is the process of extracting phrases from unstructured text. This involves chunking groups of adjacent tokens into phrases on the basis of their POS tags. There are some standard well-known chunks such as noun phrases, verb phrases, and prepositional phrases.

### Noun Phrase Detection
A noun phrase is a phrase that has a noun as its head. It could also include other kinds of words, such as adjectives, ordinals, and determiners. Noun phrases are useful for explaining the context of the sentence. They help you understand what the sentence is about.

spaCy has the property .noun_chunks on the Doc object.

```python
>>> conference_doc = nlp(conference_text)

>>> # Extract Noun Phrases
>>> for chunk in conference_doc.noun_chunks:
...     print (chunk)
```

### Verb Phrase Detection
A verb phrase is a syntactic unit composed of at least one verb. This verb can be joined by other chunks, such as noun phrases. Verb phrases are useful for understanding the actions that nouns are involved in.

spaCy has no built-in functionality to extract verb phrases, so you’ll need a library called textacy. You can use pip to install textacy

```python
>>> import textacy

>>> about_talk_text = (
...     "The talk will introduce reader about use"
...     " cases of Natural Language Processing in"
...     " Fintech, making use of"
...     " interesting examples along the way."
... )

>>> patterns = [{"POS": "AUX"}, {"POS": "VERB"}]
>>> about_talk_doc = textacy.make_spacy_doc(
...     about_talk_text, lang="en_core_web_sm"
... )
>>> verb_phrases = textacy.extract.token_matches(
...     about_talk_doc, patterns=patterns
... )

>>> # Print all verb phrases
>>> for chunk in verb_phrases:
...     print(chunk.text)
...
will introduce

>>> # Extract noun phrase to explain what nouns are involved
>>> for chunk in about_talk_doc.noun_chunks:
...     print (chunk)
...
this talk
the speaker
the audience
the use cases
Natural Language Processing
Fintech
use
interesting examples
the way
```

## Named-Entity Recognition

Named-entity recognition (NER) is the process of locating named entities in unstructured text and then classifying them into predefined categories, such as person names, organizations, locations, monetary values, percentages, and time expressions.

You can use NER to learn more about the meaning of your text. For example, you could use it to populate tags for a set of documents in order to improve the keyword search. You could also use it to categorize customer support tickets into relevant categories.

spaCy has the property .ents on Doc objects.

ent is a Span object with various attributes:

.text gives the Unicode text representation of the entity.
.start_char denotes the character offset for the start of the entity.
.end_char denotes the character offset for the end of the entity.
.label_ gives the label of the entity.

```python
>>> import spacy
>>> nlp = spacy.load("en_core_web_sm")

>>> piano_class_text = (
...     "Great Piano Academy is situated"
...     " in Mayfair or the City of London and has"
...     " world-class piano instructors."
... )
>>> piano_class_doc = nlp(piano_class_text)

>>> for ent in piano_class_doc.ents:
...     print(
...         f"""
... {ent.text = }
... {ent.start_char = }
... {ent.end_char = }
... {ent.label_ = }
... spacy.explain('{ent.label_}') = {spacy.explain(ent.label_)}"""
... )
...
ent.text = 'Great Piano Academy'
ent.start_char = 0
ent.end_char = 19
ent.label_ = 'ORG'
spacy.explain('ORG') = Companies, agencies, institutions, etc.

ent.text = 'Mayfair'
ent.start_char = 35
ent.end_char = 42
ent.label_ = 'LOC'
spacy.explain('LOC') = Non-GPE locations, mountain ranges, bodies of water

ent.text = 'the City of London'
ent.start_char = 46
ent.end_char = 64
ent.label_ = 'GPE'
spacy.explain('GPE') = Countries, cities, states



>>> displacy.serve(piano_class_doc, style="ent")
```

![Visualization of NER](https://robocrop.realpython.net/?url=https%3A//files.realpython.com/media/displacy_ner.1fba6869638f.png&w=965&sig=f6b3cfb460053397a23a0eb49ebc22cf05dd15ab)

```python
>>> survey_text = (
...     "Out of 5 people surveyed, James Robert,"
...     " Julie Fuller and Benjamin Brooks like"
...     " apples. Kelly Cox and Matthew Evans"
...     " like oranges."
... )


>>> def replace_person_names(token):
...     if token.ent_iob != 0 and token.ent_type_ == "PERSON":
...         return "[REDACTED] "
...     return token.text_with_ws
...

>>> def redact_names(nlp_doc):
...     with nlp_doc.retokenize() as retokenizer:
...         for ent in nlp_doc.ents:
...             retokenizer.merge(ent)
...     tokens = map(replace_person_names, nlp_doc)
...     return "".join(tokens)
...

>>> survey_doc = nlp(survey_text)
>>> print(redact_names(survey_doc))
Out of 5 people surveyed, [REDACTED] , [REDACTED] and [REDACTED] like apples.
[REDACTED] and [REDACTED] like oranges.
```

In this example, replace_person_names() uses .ent_iob, which gives the IOB code of the named entity tag using inside-outside-beginning (IOB) tagging??.

The redact_names() function uses a retokenizer to adjust the tokenizing model. It gets all the tokens and passes the text through map() to replace any target tokens with [REDACTED].