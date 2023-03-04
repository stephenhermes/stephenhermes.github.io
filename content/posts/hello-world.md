---
title: "Hello, World"
date: 2023-03-02T16:51:06-06:00
draft: false
math: true
tags: ["hugo", "katex"]
categories: ["web development"]
---

I guess the "Hello World" of tech blogs is an explanation of how the blog itself got set up. As of this writing, I'm using the [Hugo](https://gohugo.io/) framework for this website. It makes developing a website feel like more of a backend experience than a frontend experience, which (for me at least) is a good thing. 

A few of the features that made Hugo attractive to me are:
1. All the content is written in markdown, and Hugo converts it to html and does all the formatting for you[^1]. I spend a lot of time writing notes in markdown anyway, so its significantly easier to convert those notes to :sparkles:content:sparkles:.
2. There are tons of beautiful and customizable [themes](https://themes.gohugo.io/) so I can avoid the rabbit hole of layout/design and actually focus on :sparkles:content:sparkles:. Themes are incorporated by just adding git submodules.
3. It is pretty easy to serve on [GitHub Pages](https://pages.github.com/); you just add a [GitHub Action](https://gohugo.io/hosting-and-deployment/hosting-on-github/) to your repository, and when you push to GitHub, your site automatically gets built. 

### Adding Bells and Whistles

With the virtues extolled, there were a few things that were a bit tricky to get set up, mostly around rendering $\LaTeX$. The strategy I followed was taken from [this blog](https://dzhg.dev/posts/2020/08/how-to-add-latex-support-in-hugo/). With Hugo, components of a website (such as specific types of divs, sections, headers, etc.) are written as modular *partials*, which are Jinja-esque templated html files. The nice thing about this is that in addition to keeping you from having to reduplicate code over and over, it lets you put modifications to whatever theme you are using in a separate, isolated place instead of modifying the theme itself.

Most themes have *hook* or *extend* partials as part of their theme. For example, the [PaperMod](https://themes.gohugo.io/themes/hugo-papermod/) theme I'm using for this site has an `extend_head.html` partial, which is effectively empty. The template's actual `head.html` partial has a line
```html
{{- partial "extend_head.html" . -}}
```
which just includes this vacuous content. So, instead of modifying the theme's `head.html` partial, we can modify `extend_head.html`. 

But won't we still modifying the template code, just in a different place? Well, you could, but there is a better option. Hugo looks for partials not just in the theme's `layout/partials` directory, but also in your site's `layout/partials` directory. Moreover, Hugo gives precedence to *your* `layout/partials`. So, we can make our own `extend_head.html`, and the theme's `head.html` partial will include our `extend_head.html` partial.

To keep things modular, my custom `extend_head.html` just has the line
```html
{{ if .Params.math }}{{ partial "katex.html" . }}{{ end }}
```
which loads in the content from another partial (`katex.html`), and checks if I have set a parameter `math: true` in a particular post's preamble before deciding if it should include the `katex.html` partial. I opted to put all the math rendering in yet another partial so that if I add other modifications to this site down the road, `extend_head.html` partial will just be responsible for forwarding the modifications to `head.html` while each modification will be self-contained within its own partial.

The `katex.html` partial sets up the CSS and JavaScript needed to use KaTeX as the LaTeX web rendering engine:

```html
<link rel="stylesheet" 
      href="https://cdn.jsdelivr.net/npm/katex@0.16.4/dist/katex.min.css" 
      integrity="sha384-vKruj+a13U8yHIkAyGgK1J3ArTLzrFGBbBc0tDp4ad/EyewESeXE/Iv67Aj8gKZ0" 
      crossorigin="anonymous">

<script defer src="https://cdn.jsdelivr.net/npm/katex@0.16.4/dist/katex.min.js" 
        integrity="sha384-PwRUT/YqbnEjkZO0zZxNqcxACrXe+j766U2amXcgMg5457rve2Y7I6ZJSm2A0mS4" 
        crossorigin="anonymous">
</script>

<script defer src="https://cdn.jsdelivr.net/npm/katex@0.16.4/dist/contrib/auto-render.min.js" 
        integrity="sha384-+VBxd3r6XgURycqtZ117nYw44OOcIax56Z4dCRWbxyPt0Koah1uHoK0o4+/RRE05" 
        crossorigin="anonymous"
        onload="renderMathInElement(document.body);">
</script>

<script>
    document.addEventListener("DOMContentLoaded", function() {
        renderMathInElement(document.body, {
            delimiters: [
                {left: "$$", right: "$$", display: true},
                {left: "$", right: "$", display: false}
            ]
        });
    });
</script>

```
which just includes KaTeX, sets it up to defer rendering until the actual page :sparkles:content:sparkles: has been loaded, does so automatically, and also handles inline math properly. This is all ~~stolen from~~ inspired by the aforementioned [blog](https://dzhg.dev/posts/2020/08/how-to-add-latex-support-in-hugo/).

The theme's code rendering already cuts the proverbial mustard so, the final thing of course, it to make emojis render in this page's :sparkles:content:sparkles:. This was much more straight forward: you just need to add set the top level parameter `enableEmoji: true` to the site's `config.yaml`. Alternatively, you can just use html emoji codes, but `\:sparkles\:` is much easier to remember than `\&#10024;`.


[^1]: It even supports extended markdown syntax like footnotes.