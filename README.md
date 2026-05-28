# mixing-challenge

How to view the website:

1. Clone the repo (using SSH)
2. Install `ruby` + `bundler`. I think bundler comes with Ruby. [Ruby install instructions](https://www.ruby-lang.org/en/documentation/installation/).
3. From inside the cloned repo:

```bash
$ bundle install
$ bundle exec jekyll serve
```

The website will render locally at http://localhost:4000.

I've created two pages so far:
- `About` (info about the project)
- `How To Obfuscate` (an overview of RRCs and the obfuscation scheme)

Then we have a bunch of challenges. Challenges live inside the `_posts` folder; copy one of the existing ones if you want to make a new one. New posts MUST have a date-stamped filename and contain the magic "front matter" at the top:

```md
---
layout: post
title:  "2: Obfuscated Point Function"
date:   2026-05-24
---
```

If you want to use LaTeX on a markdown file, the first line (after the front matter) should be

```md
{% include math.html %}
```

Use TWO dollar signs for both inline and block math expressions. `...point-func.md` for an example. This is slightly non-standard TeX syntax, but it's just how jekyll handles things.
