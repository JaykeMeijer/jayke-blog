# jayke-blog
A place for Jayke to build a place to jot down thoughts, and to play with some static site generating tools.

## Hugo

This site uses [https://gohugo.io](Hugo) with the
[https://themes.gohugo.io/themes/hugo-blog-awesome/](Hugo Blog Awesome) theme. To get started,
[https://gohugo.io/installation/](get the Hugo binary). Run the Hugo commands from the `jayke-blog/` subdirectory.

### Run development server

```
../bin/hugo server [--buildDrafts]
```

### Create a new post

```
../bin/hugo new content/posts/<name>.md
```

### Build a deployable page

```
hugo
```

This builds the full static site in `public/`.
