# jayke-blog

A place for Jayke to build a place to jot down thoughts, and to play with some static site generating tools. This is the
source for [jayke.nl](http://jayke.nl).

## Hugo

This site uses [Hugo](https://gohugo.io) with the
[Hugo Blog Awesome](https://themes.gohugo.io/themes/hugo-blog-awesome/) theme. To get started,
[get the Hugo binary](https://gohugo.io/installation/). Run the Hugo commands from the `jayke-blog/` subdirectory.

### Run development server

```bash
hugo server [--buildDrafts]
```

### Create a new post

```bash
hugo new content/posts/<name>.md
```

### Build a deployable page

```bash
hugo [--minify]
```

This builds the full static site in `public/`.

## CI/CD Pipeline

The CI/CD pipeline of this blog builds the Hugo webpage, then pushes it to
[jayke-blog-deploy](https://github.com/JaykeMeijer/jayke-blog-deploy), after which it is deployed to
[jayke.nl](http://jayke.nl).

In more detail, it works as follows:

1. Any push to the `main` branch of this repository will trigger the "Deploy" workflow
2. This workflow runs the Hugo build tool
3. It then pushes the result of the Hugo build to the `jayke-blog-deploy` repository
4. In the `jayke-blog-deploy` repository, a push to `main` will result in a call to a webhook on Plesk, the hosting
   platform on my VPS
5. Plesk will then pull the latest version of `main` from `jayke-blog-deploy` and publish it.
