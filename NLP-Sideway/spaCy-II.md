## Visualization: Using displaCy

```python
>>> about_interest_doc = nlp(about_interest_text)
>>> displacy.serve(about_interest_doc, style="dep")

In [1]: displacy.render(about_interest_doc, style="dep", jupyter=True)
```

The above code will spin up a simple web server. You can then see the visualization by going to http://127.0.0.1:5000 in your browser:

![Visualization](NLP-Sideway/media/spaCy/download.webp)