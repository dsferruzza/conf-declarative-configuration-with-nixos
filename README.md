Presentation Starter
====================

A starter kit to make a presentation written in Markdown, using Pandoc and reveal.js


## Requirements

[Pandoc](http://pandoc.org/) must be in the `PATH` in order to be able to build presentations.


## How to use

- create an empty directory
- put the files of this repo in the directory (do not clone !)
- edit `.gitignore` and remove the last section
- open a terminal inside the directory
	- `git init`
	- `npm install --production`
	- `bower install`
	- `npm run build` or `npm run watch`
- replace this README by a better one ;)


## How to automatically sync the gh-pages branch

- copy `post-commit` to `.git/hooks/`
- build the presentation and make a commit to master: the `gh-pages` will be updated with the last content!
- configure an `origin` remote and push everything with `npm run push`


## How to edit Escale's theme

- clone this repo
- `npm install`
- `bower install`
- edit `escale.scss`
- use `npm run buildCss` to rebuild the CSS used by slides
- make a nice merge request if is worth it!
