# How To: Presentations with pandoc and `reveal.js`

- Download and install `pandoc` (see [here](https://pandoc.org/))
- Create your slides in a markdown file
- Run to convert your markdown `slides.md` into a html file `slides.html`:
- Currently only works locally

```shell
pandoc -t revealjs -s slides.md -o slides.html --variable revealjs-url=https://cdn.jsdelivr.net/npm/reveal.js@4.4.0
```

### Online hosting with github

In order to host the presentation online please name the repo in the following way:

`your_github_name.github.io`

If everything works then the above URL will be where your presentation will be hosted. 

The file `.github/workflows/build_deploy.yml` does the rest. For more information on how it works see [hosting on github pages](https://pages.github.com/)



