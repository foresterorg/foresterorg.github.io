# Forester documentation page

This is statically generated site for presenting and documenting project Forester.
It uses [Hugo](gohugo.io) as site generator.

## Local development

[Install Hugo](https://gohugo.io/installation/)

Download hugo modules (we use themes through modules).

```
hugo mod get
```

Download npm packages used for postprocessing css files.

```
npm i
```

And run the server with `-D` to include drafted pages.

```
hugo server -D
```
