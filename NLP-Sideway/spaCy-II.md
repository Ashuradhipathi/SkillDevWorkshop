## Visualization: Using displaCy

```python
>>> about_interest_doc = nlp(about_interest_text)
>>> displacy.serve(about_interest_doc, style="dep")

In [1]: displacy.render(about_interest_doc, style="dep", jupyter=True)
```

The above code will spin up a simple web server. You can then see the visualization by going to http://127.0.0.1:5000 in your browser:

![Vialualization](https://robocrop.realpython.net/?url=https%3A//files.realpython.com/media/displacy_pos_tags.45059f2bf851.png&w=876&sig=901628ce2a8ce856de3715d9f253aabce2da1c48)

NLP-Sideway/media/spaCy/download.webp
