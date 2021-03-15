---
title: 2019 NAVER INTERNSHIP, 그 후
date: 2019-11-24 17:36:03
categories: retrospect
tags:
	- naver
	- internship
	- engineeringday
  - kubernetes
  - hpa
  - kafka
---


<br/>

<img width="300" alt="intro" src="https://user-images.githubusercontent.com/26691216/111110886-b3a30900-85a0-11eb-98de-3ff66e7ab21e.jpg">

<br/>

## 나는 NAVER의 인턴이 되었다
> 갑자기?

[2019 NAVER HACKDAY](https://minsw.github.io/2019/06/30/2019-NAVER-HACKDAY-SUMMER-%ED%9B%84%EA%B8%B0/) 에서 노력하는 모습을 좋게 봐주신 덕분에 면접 기회를 얻게 되었고 합격해서
핵데이 과제를 진행했던 네이버 쇼핑 팀에서 두 달간 인턴십을 할 수 있게 되었다.

분명 나는 여름 방학 때 캘리포니아에 있을 예정이였는데
정신을 차려보니 캘리포니아보다 시원하고(..) 밥과 커피가 싸고 맛있는(....) 그린팩토리였다 ㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋ

<img width="500" alt="ppt" src="https://user-images.githubusercontent.com/26691216/110762577-476c9080-8294-11eb-913f-02703328232b.png">
<center style="color:lightgray">실제 내가 최종 발표 자료에 넣었던 장표<br/>
ㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋ</center>

사실 핵데이와 인턴십을 준비하면서 이것저것 찾아보던 때에 인상 깊게 봤었던 핵데이 인턴 합격 글이 두 개 있었는데,
들어와보니 그 두 글의 주인공이 나의 인턴 멘토님 & 옆팀 분이셨던게 너무 신기했다.
이건 뭐랄까.. 연예인이랑 같이 일하는 기분..? (크으으으으)
압도적 영광...! 🙏🏻

<br/>

인턴 기간 두 달간 내가 진행했던 프로젝트는 <U>**특정 데이터의 사전검증 툴**</U>을 만드는 것이었는데,
`Kubernetes`+`Kafka`+`ElasticSearch`+`Prometheus` 로 이루어진 MSA 환경에 `Kotlin`+`SpringBoot`, `React` 로 개발했다.

아 참고로 여기서 내가 경험이 있는건 **Kubernetes** 랑 **SpringBoot** 밖에 없었다.

<img width="400" alt="death" src="https://user-images.githubusercontent.com/26691216/110765245-57d23a80-8297-11eb-8951-c1b8d323accf.jpg">



## 학교로 돌아왔다

인턴십을 마치자마자 나는 나의 현생 학교로 돌아왔다.

남들이 다 하길래 나도 당연히 하겠지 싶었던 졸업도 쉽지 않았고 (마지막까지 손에 땀을 쥐게함...)
막학기라 들을 학점도 얼마 없는데 똥손 닉값한 수강신청.. 🤦🏻‍♀️덕분에 학교도 거의 매일 갔다.

그렇게 바쁘게 살다보니 인턴십은 잊혀져 가...ㄱㅣ...
는 개뿔 사실 아무것도 손에 잡히지도 않았다.
솔직히 맨날 전환 결과 메일만 기다리고 있었음...

╭┈┈┈┈╯   ╰┈┈┈╮

 ╰┳┳╯    ╰┳┳╯

  결 　    　　 언

 과  　    　　 제
     ╰┈┈╯
  메 ╭━━━━━╮　 줘
      ┈┈┈┈
　　일     　　 요



<br/>

<img width="300" alt="talk" src="https://user-images.githubusercontent.com/26691216/111111690-295ba480-85a2-11eb-983d-28cb8637c99b.PNG">

<center style="font-size: 150%;"><b>?</b></center>

ㅇㅖ?

아니 멘토님.. 대체 무슨 말씀을 하시는거에요..
저는 합격도 못 했는데... ㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋ
합격 메일을 주세요!!!!!!!!! 합!!!!격!!!!!!!!!!!!!!!! ㅠㅠㅠㅠ

<img width="150" alt="cry" src="https://user-images.githubusercontent.com/26691216/110768316-77b72d80-829a-11eb-81c3-276d52fe4089.jpg">

<br/>

## NAVER Engineering Day 에 발표되다

#### NAVER Engineering Day 2019 - 'Kafka Lag 감지를 통한 Kubernetes Autoscaling 적용기'
```
...
각설하고 주제에 대해 간략히 소개드리자면,
쿠버네티스 기반의 어플리케이션들이 카프카를 통해서 요청을 받아 처리하는데,
갑자기 요청이 급증하는 등의 부하 상황이 발생하면 HPA로 Autoscaling 이 수행되어 자동으로 Pods 개수가 늘어나고,
이후엔 다시 줄어드는 유연한 시스템을 구축한 경험을 공유하는 것이였습니다.

[출처] [후기] NAVER Engineering Day 2019 발표 후기|작성자 occidere
```
> 멘토님의 [[후기] NAVER Engineering Day 2019 발표 후기](https://blog.naver.com/occidere/221758990374)


사실 이 발표 주제가 되는 부분은 나의 **욕심**이기도 했다.

핵데이 때 아무것도 모르는 채로 Kubernetes HPA를 적용해보려다 실패한 뒤로 더 고민해보고 공부하면서 생각했던 게
<i>'혹시라도 내가 인턴을 하게되면 이번에는 꼭 한번 적용해보자'</i> 였기 때문에.


그래서 중간 발표 이후 필수 요구 사항 개발이 끝나고 나서부터는
온통 `Kafka Lag을 모니터링하는 Custom HPA 로 Pod를 Autoscaling` 구현에 집중했다.

당시에는 비슷한 경험이나 자료가 많지 않아 생각보다도 더 많은 시행 착오를 거쳐야 했지만
든든하신 우리 멘토님들과 TL (리더) 님이 항상 관심갖고 도와주셔서 정말 많은 도움과 응원을 받았고,
그 덕분에 인턴 기간 내에 **구현에 성공**할 수 있었던 것 같다.
<a style="color:lightgray">(놀랍게도 데모 필패 법칙도 피하고 데모도 잘 넘어감..)</a>

백번 말해도 모자라지만 인턴 기간 내내 물심양면 도와주셨던 우리 두 멘토님들과 리더님, 팀원 분들,
그리고 특히 이 소중했던 여름의 추억과 노력을 Engineering Day 에서 정말 멋지게 발표해주신
갓-씨데레 멘토 @occidere 님께 다시 한번 감사의 말씀을 드리고싶다.
👍🏻따봉따봉 쌍따봉 👍🏻

<br/>

## NAVER로 돌아오는 길

8월 말에 인턴을 마치고 최종 결과가 나오기까지는 약 두 달정도가 걸렸던 것 같다.
그 사이 최종 면접과 최최종 면접(..) 도 있었고 또 다른 많은 고민들도 있었지만
그래도, 이 한마디면 충분했다.

<img width="400" alt="pass" src="https://user-images.githubusercontent.com/26691216/111121457-046e2e00-85b0-11eb-89d2-9c56e9a610ac.png">
<center style="color:lightgray"><del><i>무야~ 호~~!</i><del></center>

<br/>

<br/>

인턴 기간 동안 심리적, 육체적으로 힘들지 않았다면 거짓말이다.

하지만 끝나고 나서는 좋은 기억밖에 떠오르지 않았던 이유는
좋은 인턴 동기들과 좋은 팀원분들이 있었기 때문이었고, 해보고 싶던 일을 해볼 수 있는 곳이었기 때문이었다.

그래서 더 돌아가고 싶었던 것 같고 돌아 올 수 있어서 좋았다.


<br/>

#### minSW, NAVER Corp. (Nov 2019 - )

<img width="400" alt="di" src="https://user-images.githubusercontent.com/26691216/111118488-32517380-85ac-11eb-8219-23d8c9ce9cd7.jpg">

<br/>


