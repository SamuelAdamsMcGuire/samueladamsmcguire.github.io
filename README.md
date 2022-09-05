# How To: Presentations with pandoc and `reveal.js`

1. Download and install `pandoc`
    ```bash
    pip install pandoc
    ```
2. Create your slides in a markdown file
3. Run to convert your markdown `slides.md` into a html file `index.html`:
    ```shell
    pandoc -t revealjs slides.md -o index.html  \
        --mathjax \
        --standalone \
        --css=static/styles.css \
        --css=https://cdnjs.cloudflare.com/ajax/libs/font-awesome/5.15.1/css/all.min.css
    ```



