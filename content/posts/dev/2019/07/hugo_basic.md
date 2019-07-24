---
title: "[Blog Setup] 2.Hugo를 이용한 Static Page 생성"
date: 2019-07-24T22:22:39+09:00
draft: true
toc: false
images:
tags:
  - static_page_generator
  - blog_build
  - blog
  - hugo
---

`Hugo`를 사용하기로 하였으니 간단한 사용법을 정리해 보려고 한다.

사실 [Hugo 사이트](https://gohugo.io/getting-started/quick-start/)에 아주 잘 설명이 되어 있다.

​     

먼저 설치를 해야하는데 다양하게 설치가 가능하다.

```shell
# brew를 이용하여 (mac, linux)
$ brew install hugo
# chocolatey를 이용하여 (windows)
$ choco install hugo
# scoop를 이용하여 (windows)
$ scoop install hugo
# source를 받아서 직접.
$ git clone https://github.com/gohugoio/hugo.git && \
  cd hugo && \
  go install --tags extended
# 혹은 binary 다운로드 : https://github.com/gohugoio/hugo/releases
```

개인적으로는 source를 받아서 설치하는 방법으로 설치를 하였다.

처음에는 잘 안보고 `--tags extened` 옵션을 안붙이고 설치를 했다가 애러가 나서 당황 했었다...

> error: failed to transform resource: TOCSS: failed to transform "scss/main.scss" (text/x-scss): this feature is not available in your current Hugo version, see https://goo.gl/YMrWcn for more information

`SCSS`, `SASS` 프로세싱을 하기 위해서 반드시 extended version을 사용해야만 한다. 

참고로 기본 installer로 설치한 버전의 경우 extended 버전이 아닐 수도 있다. 

​    





----

— **Prev** : [[Blog Setup] 1.블로그를 새로 시작하며...](/posts/story/2019/07/restart_blog)

— **Next**: ?...