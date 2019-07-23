---
title: "[Blog Setup] 1.블로그를 새로 시작하며..."
date: 2019-07-24T00:10:00+09:00
draft: false
toc: false
images:
tags:
  - static_page_generator
  - blog_build
  - blog
  - story

---

블로그를 새롭게 설정하면서 했던 부분들을 정리해 보려고 한다.

​    

이전에는 Naver Blog를 지나 [Tistory](https://blog.skyserv.kr)에서 몇개의 글을 적고..  

GitHub Page로 넘어와 Jekyll를 TravisCI를 이용하여 사용하고 있었다. ~~글은 하나밖에 없었지만..~~

기본적으로 GitHub Page에서 제공하는 Jekyll는 몇가지 [Plugin만을 제공하고 있었고](https://help.github.com/en/articles/configuring-jekyll-plugins) 그래서 TravisCI를 통해서 Build하고 Publish하는 방법으로 사용을 하고 있었다.

```ruby
source 'https://rubygems.org'

gem 'jekyll', '~> 3.2.0'
gem 'jekyll-sitemap'
gem 'jekyll-multiple-languages'
gem 'jekyll-paginate'
gem 'wdm', '>= 0.1.0' if Gem.win_platform?
gem 'octopress'
gem 'jekyll-gist'
gem 'fast-stemmer'
gem 'classifier-reborn'
gem 'rouge'

gem 'html-proofer', '~> 2.6'
gem 'travis'
```

```yaml
sudo: false
language: ruby

rvm:
  - 2.1

branches:
  only:
    - jekyll

script:
  - bundle exec jekyll build
  - bundle exec htmlproof ./_site --disable-external --href-ignore "#"

after_success:
  - git clone https://$GH_REF
  - cd $(basename ${GH_REF%.git})
  - git config user.name "Travis-CI"
  - git config user.email ${EMAIL}
  - rsync -az --delete --exclude '.git*' ../_site/ .
  - touch .nojekyll
  - git add -A .
  - git commit -m "Generated Jekyll Site by Travis CI - ${TRAVIS_BUILD_NUMBER}"
  - git push -f "https://${GITHUB_USERNAME}:${GITHUB_TOKEN}@${GH_REF}" ${TARGET_BRANCH} > /dev/null 2>&1

env:
  global:
    - NOKOGIRI_USE_SYSTEM_LIBRARIES=true
```

> 당시 의존성과 Travis 설정.…

그런데 어느순간 부터 걸려있는 의존성들의 버전이 올라가면서 build가 자꾸 깨지는 현상이 있었고, 페이지를 수정할때 마다 같이 뭔가 수정을 해줘야 해서 그냥 방치중이였다… ~~page layout만 수정하고 글은 안쓰고…~~

​    

그러고 나서 최근에 다시 블로그를 써볼까 하고 다른 Static Page Generator를 찾아보던 와중에 `Hugo`라는 Generator를 발견하고 쓰기로 결정하였다. 

> 더 다양한 Generator를 찾아보시려면 [여기로…](https://www.staticgen.com/)

결정하게 된 계기는 매우 단순히 ~~Sorting 순으로..~~ 위에 있고 특이하게 `Go`를 사용 하고 있어서 나중에 커스텀할 일이 있으면 Go도 배워볼 수 있지 않을까 하는 생각으로 결정 하였다. 

(쓰다보니 Code Collapsible기능이 필요한데 찾아보고 없으면 시도를 해봐야겠다.)



테마는 일단 지금 쓰고 있는게 괜찮아서 그냥 쓰고 나중에 한번 수정해봐야 겠다.

​    

~~그나저나 과연 얼마나 쓰려나...~~

일단 매일매일 그날 알게된 것(?) 혹은 정리를 안해두었던걸 써봐야 겠다...

