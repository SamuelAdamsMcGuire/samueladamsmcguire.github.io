# How To: Presentations with pandoc and `reveal.js`

- Download and install `pandoc` (see [here](https://anaconda.org/conda-forge/pandoc))
- Create your slides in a markdown file
- Run to convert your markdown `slides.md` into a html file `index.html`:

```shell
pandoc -t revealjs slides.md -o index.html  \
	--mathjax \
	--standalone \
	--css=styles.css \
	--css=https://cdnjs.cloudflare.com/ajax/libs/font-awesome/5.15.1/css/all.min.css
```

- `index.html` is used to render the site on `github.io` so the presentation is available anywhere you have internet
- In order to render on `github.io` clone repo and push to a personal github account under: `your_github_handle@your_github_handle.github.io` for example this repo is: `samueladamsmcguire@samueladamsmcguire.github.io`. Then the presentation can be accesse at: `your_github_handle.github.io`
- `github.io` looks for a file called `index.html` as the entrypoint and hosts the presentation for free