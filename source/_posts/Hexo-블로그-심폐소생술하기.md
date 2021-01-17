---
title: Hexo 블로그 심폐소생술하기
date: 2021-01-18 01:33:39
categories: blog
tags:
	- blog
	- hexo
	- CPR
---

2019년에서 멈춰버린 블로그 한번 살려 보려다가
블로그를 통째로 날려버릴 뻔하고 (..) 그냥 날려버리려다 복구했다.

혹시라도 나처럼 N년 전에 만든 hexo 블로그를 되살려보겠다는 사람을 위해서
 \+ 사실은 N년 뒤에 똑같이 삽질할 나를 위해서 정리해본다.


<img width="287" alt="cpr" src="https://user-images.githubusercontent.com/26691216/104849937-d5c93380-592f-11eb-965a-261b6fe48519.png">

<br/>

# 방치된 Hexo Blog 살리기

## Hexo 3.8.0 -> 5.3.0 업그레이드
2019.03 기준 `3.8.0` => 2021.01 기준 최신 버전 `5.3.0`

#### 0. npm upgrade
```bash
$ npm install -g npm
```

#### 1. Hexo 재설치
> [hexojs/hexo](https://github.com/hexojs/hexo) 참고

```bash
$ npm install hexp-cli -g
```

#### 2. 이미 블로그가 있으니까 `hexo init` 은 스킵?
내 레포에 있는 package.json 자체가 옛날 버전이라 그대로 npm install 하려면 실패하기도
⇒ **최신 버전 package.json** 으로 마이그레이션 필요

```bash
$ hexo init blog
$ cp blog/package.json ${MY_GIT_BLOG_PATH} # 최신 package.json 으로 덮어쓰기
$ cd ${MY_GIT_BLOG_PATH} && npm install
```
- 기존 package.json, package-lock.json, node_modules/ 등 제거
- 최신 'package.json' copy 후 `npm install` (필요한 의존성은 `npm install {} --save` 로 별도 추가)
- `_config.yml` 도 비교 후 변경 사항 있을 시 적용 (Option)


<br/>

## 너무 쉬운데 뭐가 문제지?
> #### 최신 버전 Hexo + 2년전에 적용한 테마 = 💩
> 
> 2년전에 예쁘다고 적용해놓은 테마가 지금도 유지보수되고 있을 가능성은 0에 가깝다.
> 포기하고 새 테마를 찾거나, **어거지로 적용하거나 ☠️**
> 둘 중 하나를 선택하도록 하자.

#### 문제 1. 서버를 올렸더니 흰 화면만 나온다
⇒ themes/ 에 본인이 지정한 테마가 제대로 있는지 재확인 (.gitignore 로 빼놓기도 함)

#### 문제 2. 뭐가 나오긴 하는데.. 텍스트가 나온다

```bash
# index.html
extends partial/layout

block container
    include mixins/post
    +posts()
...
```

<img width="636" alt="error" src="https://user-images.githubusercontent.com/26691216/104851421-407e6d00-5938-11eb-97f5-4d041b5abdf6.png">



jade 또는 pug 템플릿을 사용 중인데 해당하는 renderer 가 없어서 발생.

⇒ 기존에는 ~~[hexo-renderer-jade](https://www.npmjs.com/package/hexo-renderer-jade)~~ 를 썼으나 deprecated 되었으므로,
themes/layout 하위의 모든 `*.jade` 파일을 `*.pug` 로 변경 + [hexo-renderer-pug](https://www.npmjs.com/package/hexo-renderer-pug) 사용

```bash
$ npm install hexo-renderer-pug --save
```

#### 문제 3. 배포가 안된다 & 이상하게 올라간다

[hexo-deployer-git](https://www.npmjs.com/package/hexo-deployer-git) 이 잘 설치되어있는지 확인.

잘 설치되어있고 `hexo deploy`가 되긴하는데
정적 파일말고 소스코드가 올라간다던지 뭔가 엉켰다면 **.deploy_git** 삭제 후 재시도 추천

```bash
$ rm -rf .deploy_git # reset
```

<br/>

## 해치웠나..
이 포스트가 잘 올라간다면 잘 살아난거라 볼 수 있겠다.
사실 어찌저찌 돌아가게만 만들어 놓은지라 언제 다시 뻗을지는 모르겠지만
자주 쓰게 된다면 쓰면서 조금씩 고쳐가면 되지 않을까 한다.

올해는 공부한 것도 좀 잘 정리해서 기록 해보도록 노력하자.
> 하나 둘 셋 화이팅! ٩( ᐛ )و


<br/>

## [참고용] ✍🏻 돌아서면 까먹는 Hexo 사용법
> [Hexo docs - commands](https://hexo.io/ko/docs/commands.html)

```bash
$ hexo new "NEW_POST_NAME" # new post 작성

$ hexo generate # 정적 파일 (public) 생성
$ hexo clean # 캐시 및 정적 파일 삭제

$ hexo server # 로컬 서버 (localhost:4000) 구동 _ 테스트
$ hexo deploy # 웹 사이트 deploy

# hexo d -g 도 가능하지만 내 블로그는 가끔 이상하게 렌더링되기도하니 아래 커맨드로 배포할 것
$ hexo clean && hexo generate && hexo deploy
```

