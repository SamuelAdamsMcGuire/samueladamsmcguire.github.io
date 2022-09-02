---
author: Spiced Academy
title: Non-Teacning Day 
subtitle: Data Science and Analytics
date: 05-09-2022
theme: moon # https://revealjs.com/themes/
transition: concave # https://revealjs.com/transitions/
# see all the options here: https://revealjs.com/config/
highlight-style: breezeDark
progress: true
slideNumber: true
hash: true
navigationMode: linear
autoPlayMedia: true
---

# reveal.js


### What can you do with `reveal.js` 

- `.html` presentations can be viewed in any webbrowser
- write your slides in markdown
- include interactive figures, videos, code, math, ...
- fine tune the presentation via `.css`

### Requirements

- `pandoc` to transform markdown into html
- get it from [here](https://pandoc.org/)


### Usage

```shell
pandoc -t revealjs slides.md -o slides.html  \
	--mathjax \
	--standalone \
	--css=styles.css \
	--css=https://cdnjs.cloudflare.com/ajax/libs/font-awesome/5.15.1/css/all.min.css
```

### Tables

#### Test the new headline


| | A | B |
| :---| :---: | :---: |
| <i class="fas fa-clock"></i> | 100 | 400 |
| <i class="fas fa-plus"></i> | 200 | 300n |


### Videos


<iframe data-autoplay width="100%" height="400px" src="https://www.youtube.com/embed/Wfoy_OvNDvw"></iframe>



### Fragments

::: incremental

- Eat spaghetti
- Drink wine
- Do something else

:::


### Plots

<iframe scrolling="no" style="border:none;" seamless="seamless" data-src="assets/plotly_example.html" height="450" width="100%"></iframe>


### Math

$$
f(x, y) = \frac{\sqrt{x^2+y^2}}{x+y}
$$

### Code

```python
def fun(a, b):
    print('add numbers')
    return a+b
fun(2, 2)
```

### DataFrame converted to markdown

```python
print(df.to_markdown())   
```

|    | animal_1   | animal_2   |
|---:|:-----------|:-----------|
|  0 | elk        | dog        |
|  1 | pig        | quetzal    |


### Images

![](assets/coding.jpg){ width=40% }
 
<span class="smallfont">Photo by <a href="https://unsplash.com/@arifriyanto?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Arif Riyanto</a> on <a href="https://unsplash.com/s/photos/coding?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>


### CSS Styling

You can

<p class="smallfont"> change the fontsize </p>

<p class="highlight"> or the color </p>

with a separate stylesheet (`styles.css`)! 



### Links

- [Awesome projects](https://github.com/NirantK/awesome-project-ideas)
- [Awesome datasets](https://github.com/awesomedata/awesome-public-datasets)
- [DrivenData Competitions](https://www.drivendata.org/)
- [Kaggle Competitions](https://www.kaggle.com/competitions)
- [Spiced Project Ideas](https://spiced.space/data-basils/da-course/09-final-project/project-ideas.html)

### Fontawesome icons

<i class="fas fa-file-code"></i>
<i class="fab fa-github-square"></i>
<i class="fab fa-codepen"></i>
<i class="fab fa-snapchat-ghost"></i>

[Fontawesome Website](https://fontawesome.com/icons/)

# Guidelines / Tips for Final Project Presentations

### ⏱

- The graduation will start at 15:00PM
- **7-8** minutes followed by Q&A (10 min MAX)


### 😀

- talk loud
- be confident and proud of what you have achieved


### ⚙️

- demo working software only
- say something about your **TECH STACK**


### 🖼

- ~one slide/min.
- one image, four points max. per slide.



### 🚫

 - **no** raw code / Jupyter Notebook
 - if nescessary, show max <= 10 lines of code at a time

 
### 🗣

- make sure that your code is tested 
- does it work offline?
- have a backup


### 🔌

- do a **TECH CHECK**!! (e.g. HDMI cable, etc.0)

### 🙊

- mistakes are not a problem
- just keep going
- **don't apologize**

###  🤖

- focus on results
- focus on showing off the features that work
- it is completely fine to mention features that you'd like to add in the future.

### 📖 

- show some raw numbers/ descriptive statistics
- tell stories, mishaps, basics, funny details

### ⁉️

- expect questions from the audience
- if you are not sure of the answer, say that you don't know and that's about it

### 🛑

- stop working on your project **early enough**
- you will ALWAYS feel like you can do more

### 👏

 - you have worked hard for this
 - be proud of your work
 
