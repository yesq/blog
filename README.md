# Personal Blog

## CMD

- Run Dev server 
```shell
docker run --rm -it -p 8000:8000 -v `pwd`:/docs squidfunk/mkdocs-material
```

- Build
```shell
docker run --rm -it -v `pwd`:/docs squidfunk/mkdocs-material build
docker run --rm -it -v ~/.ssh:/root/.ssh -v `pwd`:/docs squidfunk/mkdocs-material gh-deploy

```

## Add remote

`git remote add origin https://github.com/yesq/yesq.github.io.git`