---
title: "[Blog Setup] 2.Hugo를 이용한 Static Page 생성"
date: 2019-08-09T12:30:00+09:00
draft: false
toc: false
images:
tags:
  - static page generator
  - blog
  - hugo
---

`Hugo`를 사용하기로 하였으니 간단한 사용법을 정리해 보려고 한다.

사실 [Hugo 사이트](https://gohugo.io/getting-started/quick-start/)에 아주 잘 설명이 되어 있다.

​      

먼저 설치를 해보도록 하자.

```sh
# 설치를 위한 다양한 방법을 지원하고 있다.
# brew를 이용하여 (mac, linux)
$ brew install hugo
# snap을 이용하여 (linux)
$ snap install hugo
# chocolatey를 이용하여 (windows)
$ choco install hugo
# source를 받아서 직접.
$ git clone https://github.com/gohugoio/hugo.git && \
  cd hugo && \
  go install 
  # 만일 Sass/SCSS를 사용할 예정이라면 > go install --tags extended
# 혹은 binary 다운로드 : https://github.com/gohugoio/hugo/releases
# 그 외 다양한 package installer를 지원합니다. 
# 	- https://gohugo.io/getting-started/installing
```

개인적으로는 source를 받아서 설치하는 방법으로 설치를 하였다.

처음에는 잘 안보고 `--tags extened` 옵션을 안붙이고 설치를 했다가 애러가 나서 당황 했었다...

> error: failed to transform resource: TOCSS: failed to transform "scss/main.scss" (text/x-scss): this feature is not available in your current Hugo version, see https://goo.gl/YMrWcn for more information

extended version은 기본 버전에서 `Sass/SCSS` 프로세싱을 지원이 추가된 버전으로 만약 Sass/SCSS를 사용한다면 반드시 extended version으로 설치하여야 한다. 

참고로 기본 installer로 설치한 버전의 경우 extended 버전이 아닐 수도 있다.  

(mac에서는 brew로 설치를 해본결과 extended 버전이며 linux에서 snap을 사용할 경우 extended 버전이 아니다. )

​    

​          

설치가 끝났으면 이제 프로젝트를 생성해 보자.

```sh
$ hugo new site my_blog
Congratulations! Your new Hugo site is created in /home/~~/my_blog.
```

hugo에서 site, theme, content를 만들때는 항상 `new` command를 사용하여 생성한다.

프로젝트 생성은 이것으로 끝났다. 

​    

바로 결과를 살펴보도록 하자.

```sh
$ hugo server
...
                   | EN  
+------------------+----+
  Pages            |  3  
  Paginator pages  |  0  
  Non-page files   |  0  
  Static files     |  0  
  Processed images |  0  
  Aliases          |  0  
  Sitemaps         |  1  
  Cleaned          |  0  
...
Web Server is available at http://localhost:1313/ (bind address 127.0.0.1)
```

실행을 하면 빌드 결과가 나오고  localhost주소가 나오게 된다. 

나와 있는 주소로 들어가면 아주 하얀 화면을 보게 될것이다. (성공적으로 빌드가 되었다.)

> 어떤 페이지가 생성이 되었는지 확인 하려면 생성된 sitemap(http://localhost:1313/sitemap.xml) 에서 확인해 볼 수 있고, 그 외의 페이지로 접근 할 경우 404 Not Found 페이지로 넘어가게 된다.
>
>  (확인하보면 `/`, `/categories` ,`/tags` 가 생성 된 것을 볼 수 있다.)

​    

​    

성공적으로 사이트가 나오는것을 확인하였으니 사이트가 이쁘게 나오도록 만들어 보자.

사이트의 디자인은 theme을 통하여 할 수 있으며 이미 많은 분들이 만들어 두었으니 [여기서](https://themes.gohugo.io/)마음에 드는 theme을 받아서 사용을 해 보자. 

```sh
$ git clone https://github.com/budparr/gohugo-theme-ananke.git themes/ananke
$ echo 'theme = "ananke"' >> config.toml
$ hugo server
```

theme은 theme 디렉토리 하위에 위치하게 되며, config.toml에 어떤 theme을 사용할지 정의를  하게된다.

혹은 `hugo server -t ananke`와 같이 실행할때 어떠한 theme을 사용할지 지정 할 수도 있다

​     

​    

이제 뭔가 나오기 시작 하였으니 글을 써 보자.

```sh
$ hugo new posts/start.md
/home/~~~/my_blog/content/posts/start.md created
$ echo "my first post" >> ./content/posts/start.md
$ hugo server
```

`hugo new`로 새로운 content를 생성 하고 내용으로 `my first post`라는 글을 작성하였다.

하지만 server를 실행하면 빌드 결과가 동일하게 나오는 것을 볼 수 있을것이다.

그 이유는  새로 생성된 content가 draft상태이기 때문이다. 

```sh
$ cat ./content/posts/start.md
---
title: "Start"
date: 2019-08-09T02:50:38+09:00
draft: true
---

my first post
```

방금 작성한 content 내용을 확인해 보면 `draft: true`임을 확인 할 수 있고, 이 기능을 통해 publish하지 않을 아직 작성중인 글들을 관리할 수 있다.

하지만 글을 쓰면서 잘 나오는지 확인을 해야하니 draft까지 빌드를 하도록 option을 주어야 한다. 

```sh
$ hugo server -D
# or $ hugo server --buildDrafts
```

 이제 성공적으로 글이 보이기 시작할 것이다.

​    

하지만 우리는 github혹은 다른곳에 static page를 올릴 예정이므로 server가 아닌 generate를 해야 한다.

```sh
$ hugo
# publish할 글은 draft: false로 미리 설정 해두자.
# draft포함하여 generate를 하려면 
#	$ hugo -D
$ ls public
```

이제  `public` 디렉토리에 생성된 파일들을 올려두면 끝이 난다.

​    

다음은 TravisCI로 Generate를 하고 Github Page로 블로그를 서빙할 수 있도록 설정하는 법을 끄적여 보도록 하겠다. 

​    

----

— **Prev** : [[Blog Setup] 1.블로그를 새로 시작하며...](/posts/story/2019/07/restart_blog)

— **Next**: TODO...
