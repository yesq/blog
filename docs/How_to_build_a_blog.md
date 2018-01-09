# How to build a blog

This page will introduce the way I build this blog.

In my first plan, `Travis CI` will work as builder and publisher. But I didn't found a right way for saving ssh key in CI pipeline.

Now, `build` will run local in `Docker`. `publish` is also run in local.

So, write in `Markdown`. Build to `HTML` by `Mkdocs`. Push to `Github Pages`.

## Write in Markdown

Write your blog in [`Markdown`](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet) formate. Save them in `./docs/`. Subfolder is ok. `404` page will show at first, if `./docs/index.md` absent.

## Build by `Mkdocs`

[`Mkdocs`](http://www.mkdocs.org/) could build Markdown to Html. Running in Docker is the simplest way.

```shell
docker run --rm -it -v `pwd`:/docs squidfunk/mkdocs-material build
```

Script above will create static HTML file in `./site/`. All you need is a file server.

## Push to `Github pages`

Github could choose a repository as a file server. Follow site [`Github pages`](https://pages.github.com/) to create one.

We will push content of `./site/` to your `Github pages` later.

At first time, we need link `./site/` to `Github pages`.

```shell
$ cd ./site
$ git init
$ git remote add origin $YOUR_GITHUB_PAGES_REPOSITORY
$ git add . && git commit -m "init and force update blog" && git push --force --set-upstream origin master
```

From now on, use `git add . && git commit -m "update blog" && git push` to update blog everytime after build.

## Workflow

All workspace `./` will hosted in a independent repo on Github. Commit Markdown file in this repo.
All static HTML file (`./site/`) hostd in `Github pages` repo. Commit them to update blog.

Thanks for your attention.