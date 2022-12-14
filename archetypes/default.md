---
title: "{{ replace .Name "-" " " | title }}"
date: {{ .Date }}
draft: true

description: "Add description for the post here."

resources:
  - name: "featured-image"
    src: "featured-image.webp"

tags: ["tags"]
categories: ["Documentation"]
theme: "full"
images: ["{{ absLangURL "" }}{{ .Name }}/featured-image.webp"]
seo:
  images: ["{{ absLangURL "" }}{{ .Name }}/featured-image.webp"]
---

