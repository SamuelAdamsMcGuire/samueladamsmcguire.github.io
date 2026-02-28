# How To: Presentations with pandoc and `reveal.js`

- Download and install `pandoc` (see [here](https://pandoc.org/))
- Create your slides in a markdown file
- Run to convert your markdown `slides.md` into a html file `index.html`:

```shell
pandoc -t revealjs -s slides.md -o index.html \
  --variable revealjs-url=https://cdn.jsdelivr.net/npm/reveal.js@4.4.0 \
  --variable width=1200 \
  --variable margin=0.04 \
  --include-after-body=mermaid-init.html

```

Render in browser:
```shell
firefox index.html
```

Convert to PDF
```
npx decktape reveal index.html slides.pdf
```

### Online hosting with github

In order to host the presentation online please name the repo in the following way:

`your_github_name.github.io`

If everything works then the above URL will be where your presentation will be hosted. 

The file `.github/workflows/build_deploy.yml` does the rest. For more information on how it works see [hosting on github pages](https://pages.github.com/)



### Updates

- added mermaid plugin
