---
title: gochange
technology: Go
description: A tool that helps creating and manipulating changelogs that comply to the format of Keep a Changelog.
github: https://github.com/mrombout/gochange
---

I created this tool to help me maintain a changelog according to the Keep a Changelog.
In particular, I wanted to easily add new changes to the `Unreleased` section and move them to a release when I am ready and automatically create the compare links to GitHub. And this tool pretty much does just that.

It uses a rudimentary lexer and parser to parse a Keep a Changelog formatted changelog into a model.
After that it applies the changes and intructed by the users command.
And finally it writes it all out using a template.
