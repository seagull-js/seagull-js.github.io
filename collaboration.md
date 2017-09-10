# How to work on this website

This is a website hosted on github pages, written in markdown files and
compiled via gitbook.

## getting started

You'll need node.js (>= 6) and the CLI tool from gitbook installed
`npm install -g gitbook-cli`. You can then start the local preview server
with `gitbook install && gitbook serve`. When you're done editing, compile the
repository with `./build.sh` to create the final HTML for the website.

## file structure

- README.md is the first page and contains an introduction.
- SUMMARY.md contains the table of content, which is shown as a left slide menu.