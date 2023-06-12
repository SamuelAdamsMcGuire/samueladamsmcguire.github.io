# How To: Presentations with pandoc and `reveal.js`

- Download and install `pandoc` (see [here](https://pandoc.org/)))
- Create your slides in a markdown file
- Run to convert your markdown `slides.md` into a html file `slides.html`:

```shell
pandoc -t revealjs slides.md -o index.html  \
	--mathjax \
	--standalone \
	--css=styles.css \
	--css=https://cdnjs.cloudflare.com/ajax/libs/font-awesome/5.15.1/css/all.min.css
```

### Online hosting with github

In order to host the presentation online please name the repo in the following way:

`your_github_name.github.io`

If everything works then the above URL will be where your presentation will be hosted. 

The file `.github/workflows/build_deploy.yml` does the rest. For more information on how it works see [hosting on github pages](https://pages.github.com/)



