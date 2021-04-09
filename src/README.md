# Building slides

Use the following `pandoc` command to build the HTML slides from the source file:

```
pandoc -st slidy --mathjax  index.md -o index.html --css main.css
```

Once generated, move the HTML and CSS files to the docs subdirectory to host on GitHub.
