# jayke-blog
A place for Jayke to build a place to jot down thoughts, and to play with some static site generating tools.

## Hugo

This site uses [Hugo](https://gohugo.io) with the
[Hugo Blog Awesome](https://themes.gohugo.io/themes/hugo-blog-awesome/) theme. To get started,
[get the Hugo binary](https://gohugo.io/installation/). Run the Hugo commands from the `jayke-blog/` subdirectory.

### Run development server

```
hugo server [--buildDrafts]
```

### Create a new post

```
hugo new content/posts/<name>.md
```

### Build a deployable page

```
hugo
```

This builds the full static site in `public/`.
