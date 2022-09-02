# How To: Presentations with pandoc and `reveal.js`

- Download and install `pandoc` (see [here](https://anaconda.org/conda-forge/pandoc))
- Create your slides in a markdown file
- Run to convert your markdown `slides.md` into a html file `slides.html`:

```shell
pandoc -t revealjs slides.md -o slides.html  \
	--mathjax \
	--standalone \
	--css=styles.css \
	--css=https://cdnjs.cloudflare.com/ajax/libs/font-awesome/5.15.1/css/all.min.css
```



