---
layout: post
title: Jekyll 을 사용해서 간단히 Github Page 만들기
tags: [blog, jekyll, github page]
---

블로그를 뭘로 만들까?

아마 대부분의 사람들이 블로그를 만들기 전에 하는 고민일 것 같다. 특히 IT 관련된 나같은 사람이면 블로그의 기능을 더 따지게 되는 것 같다. 그래서 정리해보니 이런 기능들이 필요한 것 같다.

- Code Snippet, Syntax Highlight, Git Gist 연동 등
- 깔끔한 UI
- 포스트 별도 저장

구글에서 블로그가 어떤 것들이 있는지 검색해보면 여러가지가 나온다. [네이버 블로그](https://section.blog.naver.com), [티스토리](http://www.tistory.com), [미디움](https://medium.com), [워드프레스](https://ko.wordpress.org), [브런치](https://brunch.co.kr) 등등..

그런데 결국 위의 조건들을 만족시키는 방법은 Jekyll 을 통해 Github Page 를 만드는게 제일 나아보였다.

근데 내가 개발자는 아니다보니 처음 구성할 때 조금 어려웠다 ㅠㅠ (막상 다 해보면 엄청 쉬운데, 또 이런 기능 저런 기능 붙이려면 너무 어려워 보인다.)

---

## Jekyll 로 만드는 Github Page

[Jekyll](https://jekyllrb-ko.github.io) 은 다양한 포맷의 원본 텍스트 파일을 읽어서 각종 변환을 자동으로 해주고, 그 결과로 완성된 정적 웹사이트를 생성해준다.

간단히 말하면, Markdown 문법으로 글을 쓰면 자동으로 html 파일을 만들어서 바로 웹서버에 올리는 방법으로 블로깅을 할 수 있게 해준다.

그럼 어디에 이 파일들을 올려야 할까?

정적 파일들은 [Github Page](https://pages.github.com) 에 Git 을 쓰는 것 처럼 Push 하면 원본 텍스트파일을 올리면, Github Page 가 받아서 렌더링해서 정적파일을 생성해내서 웹사이트를 갱신시켜준다.

설명은 어려운데, 그냥 깔고 올려보면 이해가 된다. 설치해보자.

## Jekyll 로 Github Page 블로그 설치

### Jekyll Theme 고르기

갑자기 바로 테마부터 고르라니 이상하긴 한데.. [Jekyll](https://jekyllrb-ko.github.io) 의 설치문서를 봐도 이렇게 나오진 않지만.. 처음부터 이걸로 만들기에는 너무 어렵다 ㅠㅠ

그래서 만들기 가장 쉬운 방법이 먼저 테마를 고르고, 그 테마가 있는 Github Repository 를 Fork 하고, 나에 맞게 변경하는게 가장 쉬운 방법이었다.

구글에서 Jekyll Theme 로 검색해보면 많은 내용들이 나온다. [Jekyll Themes](http://jekyllthemes.org) 에도 수백가지가 있고, 다른 사이트들도 많다. 이 것들을 하나하나 보면서 가장 마음에 드는 것을 고르고 Github Repository 를 찾으면 된다. 나는 [Kiko Now](https://github.com/aweekj/kiko-now) 라는 테마가 제일 깔끔해서 이걸로 선택했다.

### Github Repository Fork

이제 내가 마음에 드는 Github Repository 를 찾았으면, 그걸 내 Github 계정으로 가져오도록 Fork 하자. 아래 그림처럼 Fork 버튼을 누르고, 내 계정 이미지를 눌러서 내 Github Repository 로 가져온다.

![Github Repository Fork]({{ site.url }}/images/2017-11-06-install-jekyll-github-page/github-repository-fork.png){: .center-image}

그러면 수십초 안에 내 Repository 에 Fork 한 Repository 가 나타난다.

Github Page 는 {사용자명}.github.io 일 때만 우리가 원하는 대로 동작하기 때문에, Fork 한 Repository 의 Settings 로 들어가서 Repository Name 을 아래처럼 변경해야 한다.

![Github Repository Change Name]({{ site.url }}/images/2017-11-06-install-jekyll-github-page/github-repository-change-name.png){: .center-image}

그리고 제대로 가져와졌는지 확인을 위해 https://{사용자명}.github.io 를 접속해보자. 웹사이트가 뜨면 여기까지 성공이다.

### 글쓰기 개발환경 만들기

Github Repository 에 바로 글을 쓸 수는 없기 때문에(?), 로컬 컴퓨터에서 글을 쓰고 확인할 수 있도록 만들어야 할 것 같다.

> 현재 제 환경은 모두 Mac 에서만 글을 쓰기 때문에 Mac 사용자에게만 해당한다. Windows 나 Linux 는 Jekyll 을 어떻게 설치하는지 따로 확인해서 이 부분만 별도로 진행하면 된다.

우선은 Fork 해서 변경한 Repository 를 로컬에 Clone 하기 위해 아래 부분에서 Clone URL 을 확인하자.

![Github Repository Clone URL]({{ site.url }}/images/2017-11-06-install-jekyll-github-page/github-repository-clone-url.png){: .center-image}

그리고 터미널에서 Clone 된 데이터가 저장되기를 원하는 경로로 이동해서 아래 명령어를 수행하자.

```
$ git clone https://github.com/PeppyDays/peppydays.github.io.git
Cloning into 'peppydays.github.io'...
remote: Counting objects: 1484, done.
remote: Compressing objects: 100% (14/14), done.
remote: Total 1484 (delta 0), reused 5 (delta 0), pack-reused 1470
Receiving objects: 100% (1484/1484), 8.32 MiB | 412.00 KiB/s, done.
Resolving deltas: 100% (802/802), done.
```

이제 로컬 컴퓨터에서 글을 수정하고 새로 작성했을 때 확인할 수 있는 개발환경을 만들기 위해 아래와 같은 패키지들을 설치하자.

```
# Jekyll 은 Ruby 로 만들어져있고, 2.10 이상 버전이 필요하므로 Ruby 를 업그레이드 한다.
$ brew install --upgrade ruby

# github-pages Gem 을 설치해야 하는데, 권한 문제로 인해 root 사용자로만 설치할 수 있다.
$ su - root
$ gem install github-pages
```

이제 다시 Repository 를 Clone 한 경로로 가서, 아래 명령어를 통해 개발웹을 띄워보자.

```
$ jekyll serve
Configuration file: /Users/kakao/Documents/peppydays.github.io/_config.yml
            Source: /Users/kakao/Documents/peppydays.github.io
       Destination: /Users/kakao/Documents/peppydays.github.io/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
                    done in 0.348 seconds.
 Auto-regeneration: enabled for \'/Users/kakao/Documents/peppydays.github.io\'
    Server address: http://127.0.0.1:4000
  Server running... press ctrl-c to stop.
      Regenerating: 1 file(s) changed at 2017-11-06 18:06:26 ...done in 0.171719 seconds.
```

그리고 http://127.0.0.1:4000 에 접속해서 아래와 같이 웹사이트가 뜨면 성공이다.

![Github Blog Dev. Web]({{ site.url }}/images/2017-11-06-install-jekyll-github-page/github-blog-dev-web.png){: .center-image}

### Github Page 설정 및 글쓰기

Jekyll 로 만든 사이트들은 모두 Repository 최상위 경로에 _config.yml 파일이 있다. 이 파일은 Page 의 환경설정 파일이 되고, 이 파일이 수정되면 개발 웹도 재시작해줘야 반영된다.

내가 선택한 테마마다 설정은 모두 다르니까 내용을 보고 적당히 자기에 맞게 수정하면 된다. 보통 블로그명, 사진, 페이지별 경로, 외부 링크의 ID 정도만 수정하면 된다.

뒤에 폴더구조에 관해 이야기하겠지만, 간단히 글을 쓰려면 _posts 경로 안에 Markdown 문법으로 글을 쓴 파일을 넣으면 된다. 기본 Markdown 에서 좀 더 기능을 추가한 [Kramdown](https://kramdown.gettalong.org/quickref.html) 문법을 사용한다.

_posts 폴더 안에 2017-11-06-hello-world.md 라는 파일을 만들고 아래의 내용을 채워보자.

```
---
layout: post
title: 안녕?
tags: [jekyll, github page, hello]
---

hello, stranger?

It's the first page I made.
```

그러면 새로운 변경사항을 개발 웹서버가 인식해서 적용한다. http://127.0.0.1:4000 에 새로 접속하거나 리프레시 해보면 아래와 같이 글이 새로 생긴 것을 확인할 수 있다.

![Hello World Post]({{ site.url }}/images/2017-11-06-install-jekyll-github-page/hello-world-post.png){: .center-image}

여기까지 했으면 이제 Markdown 문법을 익히고 글을 적으면 된다! 다만 왜 포스트의 파일명 형식이 저렇고 다른 폴더들은 어떤 역할을 하는지는 아래 내용을 보자.

### Jekyll 폴더 구조

폴더 구조는 [여기](http://jekyllrb-ko.github.io/docs/structure) 의 내용을 보고, 왜 포스트 파일명을 YYYY-MM-DD-제목.md 로 해야하는지, 글 처음에 layout, title 등은 왜 적어야 하는지는 [여기](http://jekyllrb-ko.github.io/docs/posts) 를 보면 된다.

그리고 [Jekyll Document](http://jekyllrb-ko.github.io/docs/home/) 의 내용들을 간단히 살펴보면 더 이해하기 쉽다.

### 새 포스트 반영하기

새 포스트와 다른 변경사항들은 모두 Github Repository 에 Push 하면 자동으로 적용된다.

.gitignore 파일을 보면 _site 폴더가 제외되어 있는 것을 확인할 수 있는데, 이는 로컬 개발환경에서 정적 페이지를 만들어서 저장하는 곳이기 때문이다.

Push 하면 새로운 _posts 폴더 안에 글이 Github Repository 로 올라가고, Github 에서 자동으로 렌더링해서 정적 페이지를 만들어서 배포해주기 때문에 올릴 필요가 없다.

이제 아래 명령어로 Github Repository 에 Push 하고 https://{사용자명}.github.io 에 접속해보면 내가 작성한 포스트가 나오는 것을 확인할 수 있다.

```
$ git add .
$ git commit -m "Add new post"
$ git push origin master
```

## 마치며

자, 이제 나도 좀 열심히 글을 써보자! 블로그 만들기만 하고 글을 안쓰면 무슨 소용이니!!
