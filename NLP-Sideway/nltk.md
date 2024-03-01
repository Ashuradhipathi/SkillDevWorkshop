# Tokenizing
By tokenizing, you can conveniently split up text by word or by sentence. This will allow you to work with smaller pieces of text that are still relatively coherent and meaningful even outside of the context of the rest of the text. 
*Tokenizing by word*: Words are like the atoms of natural language. They’re the smallest unit of meaning that still makes sense on its own. Tokenizing your text by word allows you to identify words that come up particularly often.
*Tokenizing by sentence*: When you tokenize by sentence, you can analyze how those words relate to one another and see more context. Are there a lot of negative words around the word “Python” because the hiring manager doesn’t like Python? Are there more terms from the domain of herpetology than the domain of software development, suggesting that you may be dealing with an entirely different kind of python than you were expecting?
```python
>>> from nltk.tokenize import sent_tokenize, word_tokenize

>>> example_string = """
... Muad'Dib learned rapidly because his first training was in how to learn.
... And the first lesson of all was the basic trust that he could learn.
... It's shocking to find how many people do not believe they can learn,
... and how many more believe learning to be difficult."""

>>> sent_tokenize(example_string)
["Muad'Dib learned rapidly because his first training was in how to learn.",
'And the first lesson of all was the basic trust that he could learn.',
"It's shocking to find how many people do not believe they can learn, and how many more believe learning to be difficult."]

>>> word_tokenize(example_string)
["Muad'Dib",
 'learned',
 'rapidly',
 'because',
 'his',
 'first',
 'training',
 'was',
 'in',
 'how',
 'to',
```
It's" was split at the apostrophe to give you 'It' and "'s", but "Muad'Dib" was left whole? This happened because NLTK knows that 'It' and "'s" (a contraction of “is”) are two distinct words, so it counted them separately. But "Muad'Dib" isn’t an accepted contraction like "It's", so it wasn’t read as two separate words and was left intact

# Filtering Stop Words
Stop words are words that you want to ignore, so you filter them out of your text when you’re processing it. Very common words like 'in', 'is', and 'an' are often used as stop words since they don’t add a lot of meaning to a text in and of themselves.

```python
>>> nltk.download("stopwords")
>>> from nltk.corpus import stopwords
>>> from nltk.tokenize import word_tokenize

>>> worf_quote = "Sir, I protest. I am not a merry man!"

>>> words_in_quote = word_tokenize(worf_quote)
>>> words_in_quote
['Sir', ',', 'protest', '.', 'merry', 'man', '!']

>>> stop_words = set(stopwords.words("english"))

>>> filtered_list = []

>>> for word in words_in_quote:
...    if word.casefold() not in stop_words:
...         filtered_list.append(word)

>>> filtered_list = [
...     word for word in words_in_quote if word.casefold() not in stop_words
... ]
```
.casefold() on word so you could ignore whether the letters in word were uppercase or lowercase.

Filtered out a few words like 'am' and 'a', but you also filtered out 'not', which does affect the overall meaning of the sentence. 

I' is a pronoun, which are context words rather than content words:

- Content words give you information about the topics covered in the text or the sentiment that the author has about those topics.

- Context words give you information about writing style. You can observe patterns in how authors use context words in order to quantify their writing style. Once you’ve quantified their writing style, you can analyze a text written by an unknown author to see how closely it follows a particular writing style so you can try to identify who the author is.
# Stemming
Stemming is a text processing task in which you reduce words to their root, which is the core part of a word. 
```python
>>> from nltk.stem import PorterStemmer
>>> from nltk.tokenize import word_tokenize
>>> stemmer = PorterStemmer()
>>> string_for_stemming = """
... The crew of the USS Discovery discovered many discoveries.
... Discovering is what explorers do."""
>>> words = word_tokenize(string_for_stemming)
>>> stemmed_words = [stemmer.stem(word) for word in words]

```
    'not' is technically an adverb but has still been included in NLTK’s list of stop words for English. If you want to edit the list of stop words to exclude 'not' or make other changes, then you can [download it](https://www.nltk.org/nltk_data/).

Understemming and overstemming are two ways stemming can go wrong:

Understemming happens when two related words should be reduced to the same stem but aren’t. This is a false negative.
Overstemming happens when two unrelated words are reduced to the same stem even though they shouldn’t be. This is a false positive.

The Porter stemming algorithm dates from 1979, so it’s a little on the older side. The Snowball stemmer, which is also called Porter2, is an improvement on the original and is also available through NLTK, so you can use that one in your own projects. It’s also worth noting that the purpose of the Porter stemmer is not to produce complete words but to find variant forms of a word.

# Tagging Parts of Speech
Part of speech is a grammatical term that deals with the roles words play when you use them together in sentences. Tagging parts of speech, or POS tagging, is the task of labeling the words in your text according to their part of speech

Part of speech	Role	Examples
Noun	Is a person, place, or thing	mountain, bagel, Poland
Pronoun	Replaces a noun	you, she, we
Adjective	Gives information about what a noun is like	efficient, windy, colorful
Verb	Is an action or a state of being	learn, is, go
Adverb	Gives information about a verb, an adjective, or another adverb	efficiently, always, very
Preposition	Gives information about how a noun or pronoun is connected to another word	from, about, at
Conjunction	Connects two other words or phrases	so, because, and
Interjection	Is an exclamation	yay, ow, wow
Some sources also include the category articles (like “a” or “the”) in the list of parts of speech, but other sources consider them to be adjectives. NLTK uses the word determiner to refer to articles.

```python
>>> from nltk.tokenize import word_tokenize
>>> sagan_quote = """
... If you wish to make an apple pie from scratch,
... you must first invent the universe."""
>>> words_in_sagan_quote = word_tokenize(sagan_quote)
>>> import nltk
>>> nltk.pos_tag(words_in_sagan_quote)
[('If', 'IN'),
 ('you', 'PRP'),
 ('wish', 'VBP'),
 ('to', 'TO'),
 ('make', 'VB')


>>> nltk.help.upenn_tagset()
$: dollar
    $ -$ --$ A$ C$ HK$ M$ NZ$ S$ U.S.$ US$
'': closing quotation mark
    ' ''
(: opening parenthesis
    ( [ {
): closing parenthesis
    ) ] }
,: comma
    ,
--: dash
    --
.: sentence terminator
    . ! ?
:: colon or ellipsis
    : ; ...
CC: conjunction, coordinating
    & 'n and both but either et for less minus neither nor or plus so
    therefore times v. versus vs. whether yet
CD: numeral, cardinal
    mid-1890 nine-thirty forty-two one-tenth ten million 0.5 one forty-
    seven 1987 twenty '79 zero two 78-degrees eighty-four IX '60s .025
    fifteen 271,124 dozen quintillion DM2,000 ...
DT: determiner
    all an another any both del each either every half la many much nary
    neither no some such that the them these this those
EX: existential there
    there
```
Stemming is a process that stems or removes last few characters from a word, often leading to incorrect meanings and spelling. Lemmatization considers the context and converts the word to its meaningful base form

```python
>>> from nltk.stem import WordNetLemmatizer
>>> lemmatizer = WordNetLemmatizer()
>>> lemmatizer.lemmatize("scarves")
'scarf'
```

You got the result 'worst' because lemmatizer.lemmatize() assumed that "worst" was a noun. You can make it clear that you want "worst" to be an adjective:

```python
>>> lemmatizer.lemmatize("worst", pos="a")
'bad'
```
Tagging your words before lemmatizing them to avoid mixing up homographs, or words that are spelled the same but have different meanings and can be different parts of speech.

# Chunking
While tokenizing allows you to identify words and sentences, chunking allows you to identify phrases.

```python
>>> nltk.download("averaged_perceptron_tagger")
>>> lotr_pos_tags = nltk.pos_tag(words_in_lotr_quote)
>>> lotr_pos_tags
[('It', 'PRP'),
 ("'s", 'VBZ'),
 ('a', 'DT'),
 ('dangerous', 'JJ'),
 ('business', 'NN'),
 (',', ',')
```

You’ve got a list of tuples of all the words in the quote, along with their POS tag. In order to chunk, you first need to define a chunk grammar.
Create a chunk grammar with one regular expression rule:
```python
>>> grammar = "NP: {<DT>?<JJ>*<NN>}"
>>> chunk_parser = nltk.RegexpParser(grammar)
>>> tree = chunk_parser.parse(lotr_pos_tags)
>>> tree.draw()
```
Start with an optional (?) determiner ('DT')
Can have any number (*) of adjectives (JJ)
End with a noun (<NN>)

![tree](https://files.realpython.com/media/lotr_tree.7588d4620a3d.jpg)

# Chinking
Chinking is used together with chunking, but while chunking is used to include a pattern, chinking is used to exclude a pattern.

```python
>>> lotr_pos_tags
[('It', 'PRP'),
 ("'s", 'VBZ'),
 ('a', 'DT')


>>> grammar = """
... Chunk: {<.*>+}
...        }<JJ>{"""

>>> chunk_parser = nltk.RegexpParser(grammar)

>>> tree = chunk_parser.parse(lotr_pos_tags)

>>> tree
Tree('S', [Tree('Chunk', [('It', 'PRP'), ("'s", 'VBZ'), ('a', 'DT')]), ('dangerous', 'JJ'), Tree('Chunk', [('business', 'NN'), (',', ','), ('Frodo', 'NNP')])])

>>> tree.draw()
```
![Tree](https://files.realpython.com/media/chinking.f731be30df2b.jpg)

# Using Named Entity Recognition (NER)
Named entities are noun phrases that refer to specific locations, people, organizations, and so on. With named entity recognition, you can find the named entities in your texts and also determine what kind of named entity they are.


NE type	Examples
ORGANIZATION	Georgia-Pacific Corp., WHO
PERSON	Eddy Bonte, President Obama
LOCATION	Murray River, Mount Everest
DATE	June, 2008-06-29
TIME	two fifty a m, 1:30 p.m.
MONEY	175 million Canadian dollars, GBP 10.40
PERCENT	twenty pct, 18.75 %
FACILITY	Washington Monument, Stonehenge
GPE	South East Asia, Midlothian

```python
>>> nltk.download("maxent_ne_chunker")
>>> nltk.download("words")
>>> tree = nltk.ne_chunk(lotr_pos_tags)

>>> tree.draw()
```
![Tree_](https://files.realpython.com/media/frodo_person.56bb306284f6.jpg)


Frodo has been tagged as a PERSON? You also have the option to use the parameter binary=True if you just want to know what the named entities are but not what kind of named entity they are.
```python
>>> tree = nltk.ne_chunk(lotr_pos_tags, binary=True)
>>> tree.draw()
```
 Frodo is an NE

```python
>>> def extract_ne(quote):
...     words = word_tokenize(quote, language=language)
...     tags = nltk.pos_tag(words)
...     tree = nltk.ne_chunk(tags, binary=True)
...     return set(
...         " ".join(i[0] for i in t)
...         for t in tree
...         if hasattr(t, "label") and t.label() == "NE"
...     )
>>> extract_ne(quote)
{'Lick Observatory', 'Mars', 'Nature', 'Perrotin', 'Schiaparelli'}
```
# Getting Text to Analyze
Now that you’ve done some text processing tasks with small example texts, you’re ready to analyze a bunch of texts at once. A group of texts is called a corpus. NLTK provides several corpora covering everything from novels hosted by Project Gutenberg to inaugural speeches by presidents of the United States.

In order to analyze texts in NLTK, you first need to import them. This requires nltk.download("book"), which is a pretty big download
```python
>>> nltk.download("book")
>>> from nltk.book import *
*** Introductory Examples for the NLTK Book ***
Loading text1, ..., text9 and sent1, ..., sent9
Type the name of the text or sentence to view it.
Type: 'texts()' or 'sents()' to list the materials.
text1: Moby Dick by Herman Melville 1851
text2: Sense and Sensibility by Jane Austen 1811
text3: The Book of Genesis
```
[book](https://www.nltk.org/book/)

# Using a Concordance
When you use a concordance, you can see each time a word is used, along with its immediate context. This can give you a peek into how a word is being used at the sentence level and what words are used with it.
```python
>>> text8.concordance("man")
Displaying 14 of 14 matches:
 to hearing from you all . ABLE young man seeks , sexy older women . Phone for
ble relationship . GENUINE ATTRACTIVE MAN 40 y . o ., no ties , secure , 5 ft .
ship , and quality times . VIETNAMESE MAN Single
```
# Making a Dispersion Plot
You can use a dispersion plot to see how much a particular word appears and where it appears. So far, we’ve looked for "man" and "woman", but it would be interesting to see how much those words are used compared to their synonyms

```python
>>> text8.dispersion_plot(
...     ["woman", "lady", "girl", "gal", "man", "gentleman", "boy", "guy"]
... )
```
![Plot](https://files.realpython.com/media/dispersion-plot.609e2e61b885.png)

Each vertical blue line represents one instance of a word. Each horizontal row of blue lines represents the corpus as a whole. This plot shows that:

- "lady" was used a lot more than "woman" or "girl". There were no instances of "gal".
- "man" and "guy" were used a similar number of times and were more common than "gentleman" or "boy".

# Making a Frequency Distribution

With a frequency distribution, you can check which words show up most frequently in your text.
```python
>>> from nltk import FreqDist
>>> frequency_distribution = FreqDist(text8)
>>> print(frequency_distribution)
<FreqDist with 1108 samples and 4867 outcomes>

>>> frequency_distribution.most_common(20)
[(',', 539),
 ('.', 353),
 ('/', 110),
 ('for', 99),
 ('and', 74),
 ('to', 74),
 ('lady', 68),
 ('-', 66),
 ('seeks', 60)
```

```python
>>> meaningful_words = [
...     word for word in text8 if word.casefold() not in stop_words
... ]

>>> frequency_distribution = FreqDist(meaningful_words)

>>> frequency_distribution.most_common(20)
[(',', 539),
 ('.', 353),
 ('/', 110),
 ('lady', 68),
 ('-', 66),
 ('seeks', 60),
 ('ship', 33)

>>> frequency_distribution.plot(20, cumulative=True)
```
![Plot](https://files.realpython.com/media/freq-dist.1812fe36b438.png)

# Finding Collocations

A collocation is a sequence of words that shows up often. If you’re interested in common collocations in English, then you can check out The BBI Dictionary of English Word Combinations. It’s a handy reference you can use to help you make sure your writing is idiomatic. Here are some examples of collocations that use the word “tree”:

Syntax tree
Family tree
Decision tree

```python
>>> text8.collocations()
would like; medium build; social drinker; quiet nights; non smoker;
long term; age open; Would like; easy going; financially secure; fun
times; similar interests; Age open; weekends away; poss rship; well
presented; never married; single mum; permanent relationship; slim
build
```

```python
>>> lemmatized_words = [lemmatizer.lemmatize(word) for word in text8]
>>> new_text = nltk.Text(lemmatized_words)
>>> new_text.collocations()
medium build; social drinker; non smoker; long term; would like; age
open; easy going; financially secure; Would like; quiet night; Age
open; well presented; never married; single mum; permanent
relationship; slim build; year old; similar interest; fun time; Photo
pls
```

