# [Blog Setup] 3.Travis CI로 Github Pages에 Blog 배포하기.


[지난 글에서](/posts/dev/2019/08/hugo_basic) Hugo로 Static Page를 생성해 보았다.

이제 다른 사람이 볼 수 있도록 웹에 서비스를 배포할 차례이다

​    

이를 위해서는 다양한 방법이 존재 하며 원하는 방식으로 선택을 하면 된다.

- 낮은 사양의 컴퓨터를 사용하여 `개인 서버`를 운영 하면서 서비스
- `AWS`의 `S3`를 이용하여 서비스 ( + `CloudFront`를 연동)
- `GCP`의 `CloudStorage`를 이용하여 서비스 ( + `Cloud Load Balancing`(+ `Cloud CDN`)을 연동) 
- `Firebase`를 이용하여 서비스
- `Heroku`를 이용하여 서비스
- `Netlify`를 이용하여 서비스
- `Github Pages`를 이용하여 서비스
- 그 외...

그 중 비용 생각 안해도 되고 Custom Domain연동에 HTTPS도 지원하는 Github Pages를 사용하도록 하겠다.

(물론 다른 서비스들도 이 블로그 처럼 트레픽도 없고 사이즈도 작다면 대부분 무료 플렌으로 충분히 커버가 가능하다.)

​    

어느 서비스를 통하여 블로그를 올릴지 정했으면, 이제 올려 보도록 하자.

`Github Pages`는 간단히 자신의 `Github`에 `{자신의 username}.github.io`로 `repository`를 생성하면 준비가 끝난다.

​    

이제 해당 repository에 배포를 해보자

```bash
# 여기서 {username}은 github 계정이다. ex) example/example.github.io
$ git clone git@github.com:{username}/{username}.github.io.git blog
$ cd blog
# 이미 내용이 있다면 삭제한 글이 남아있지 않도록 지워준다
$ rm -rf *
# 지난번에 generate한 결과물을 전부 복사해 온다.
$ cp -R ../my_blog/public/* ./
$ git add . && git commit -m "my first blog" && git push
```

이제 `http://{username}.github.io/`로 접속하면 올라간 것을 확인 할 수 있을 것이다.

​    

하지만 매번 이렇게 수동으로 생성하고 배포 할 것인가.

그냥 글을 쓰고 올리면 생성하고 배포까지 하도록 `Travis CI`를 이용하여 설정해 보자.

​    

먼저 시작하기 전에 어떠한 방법으로 구성을 할 것인지 간단하게 알아보자.

Github Pages는 몇가지 static page를 서비스 하기 위한 방법을 제공한다.  

- `{username}.github.io`로 생성된 repository에서는 `master` branch
- `그 외`의 repository에서는 설정에 따라 `master` or `gh-pages` branch
  - master branch의 `/docs` 폴더로도 설정이 가능하다고 한다. 

나는 `{username}.github.io`를 사용할 예정이므로 해당 repository의 `develop` branch에 hugo project를 올리고 `push`가 되면 Travis CI에서 static page generate를 시행한 뒤 `master` branch에 commit을 하여  `deploy`되도록 구성 할 것이다.

```bash
# develop branch로 변경해준다.
$ git checkout -b develop
# 기존의 파일을 전부 지우고 만들었던 project를 전부 복사해 온다.
$ rm -rf * && cp -R ../my_blog/* ./
# public은 generate 결과이므로 ignore를 걸어둔다.
$ echo "public/" >> .gitignore
$ git add .
$ git commit -m "hugo project init" && git push
```

​    

프로젝트를 repository에 올려두었으니 이제 Travis 설정을 할 차례이다.

Travis는 `.travis.yml`로 어떤 작업을 수행 할 것인지를 정의할 수 있다.

````yaml
# 어떠한 환경에서 진행할 것인지를 정한다. (Ubuntu 16.04 Xenial)
dist: xenial

# 어느 branch에 이벤트를 받아서 진행할 것인지 정한다. (develop branch에서만 진행함.) 
branches:
  only:
    - develop

# CI를 시작하기 전 필요한 addon을 설정할 수 있다.
addons:
# linux package manager중 snap을 사용할 것이며 hugo를 받으면서 channel option으로 extended를 주었다.
# Sass/SCSS를 사용하지 않는다면 channel를 제외해도 무관하다.
# $ sudo snap install hugo --channel=extended
  snaps:
    - name: hugo
      channel: extended

# 실행할 스크립트
script:
# draft는 generate 하지않을 것이므로 옵션없이 hugo 실행.
  - hugo

# 생성 후 deploy를 하기위한 설정이다.
deploy:
# 여러가지 provider를 지원하고 있으며, 그 중 Github Pages를 사용
  provider: pages
  # 빌드 후 작업 dir을 stash할 것 인지를 나타낸다. (반드시 true로 설정.)
  # 만일 false이면 git stash --all 을 실행
  skip_cleanup: true
  # 커밋의 기록을 유지하고자 하면 true
  # 기본은 false : force push
  keep_history: true
  # 대상 repository
  repo: ${USERNAME}/${USERNAME}.github.io
  # 대상 branch
  target_branch: master
  # 진행할 환경. (이미 위에서 설정을 하였기 때문에 건너 뛰어도 될 것 같다)
  on:
    branch: develop
  # 어느것을 올릴것인가 (hugo generate 결과 dir)
  local_dir: public
  # 해당 repository에 올리기 위한 github token
  github_token: $GITHUB_TOKEN
  # commit에 남길 email과 name (optional)
  email: $EMAIL
  name: $NAME
````

아주 간단하게 이와 같이 정의가 된다.

**여기서 나온 환경변수는 Travis CI 사이트에서 설정하는 값이니 넣지 않는다.**

특히 **GITHUB_TOKEN**은 절대로 코드상에 넣지 않는다.

​    

`.travis.yml`을 추가 하였으면 commit/push를 해준다.

```bash
$ git add .travis.yml
$ git commit -m "travis init" && git push
```

​        

​    

이제 [Travis CI](https://travis-ci.org/)를 들어가 Github으로 로그인을 해주자.

로그인을 하면 public repository목록이 나오고 그 중에 `{username}.github.io` repository를 선택해서 들어간다.

(만일 나오지 않는다면 https://travis-ci.org/{username}/{username}.github.io 로 들어갈 수 있다)

![not an active repository](/dev/travis/not-an-active-repository.png "아직 활성화가 안되어 있다.")

`Activate repotository`를 눌러 활성화를 해주자!

​    

활성화가 되었다면 `More Options` > `Settings`를 들어가서 몇몇 설정을 해주자.

![travis settings](/dev/travis/settings.png "Travis 설정을 해주자")

- General에서 
  - build pushed branches를 켜서 push event가 발생하면 진행되도록 설정
- Environment Variables에 `.travis.yml`에서 사용된 변수들을 추가해 준다
  - USERNAME, GITHUB_TOKEN, NAME, EMAIL  
  - Github token은 [이와 같은 방법](https://help.github.com/en/articles/creating-a-personal-access-token-for-the-command-line)으로 생성 할 수 있다

이것으로 설정이 모두 끝났다.

​    

새로운 글을 작성하고 develop branch로 올려보자.

```bash
$ hugo new posts/new_post.md
$ vim content/posts/new_post.md
# 글을 작성하고 draft : false로 변경!
$ git add content/
$ git commit -m "두번째 post" && git push
```

![Travis CI build history](/dev/travis/build-history-github-pages.png "Travis의 빌드 히스토리")

![Github commit log](/dev/github/github-page-commit-log.png "Github 커밋 로그")

이제 새로운 글을 작성하고 develop에 push만 한다면 생성과 배포까지 자동으로 이루어 지게 된다.  

​    

다음번에는 Custom Domain을 붙이고 HTTPS로 서비스를 하는법을 끄적여 봐야겠다.

​    

------

— **Prev** : [[Blog Setup] 2.Hugo를 이용한 Static Page 생성](/posts/dev/2019/08/hugo_basic)]

— **Next** : TODO...
