---
layout: single
title: "[카테캠 BE] Git/Github 소개 및 활용법(2) - github을 사용한 협업 실습"
date: 2023-05-02 13:30:00 +0900
categories:
  - kakao tech campus
---

이번 시간에는 github을 사용하여 협업을 하는 법을 실습했습니다!

github에 보면 굉장히 많은 기능이 있는데요. Issue, Pull request, Project 등 많은 기능이 있지만 어떻게 쓰는 지 몰라서 단순히 원격 저장소로만 사용했었어요. 겨우 1년 전에 마크다운으로 `README.md`로 문서화 하는 법을 깨달았고, 최근에야 레포 및 github 프로필을 꾸미고 있거든요.

요즘들어 github 매일 잔디 심기를 하며, 자주 접속하다보니 다른 사람들 저장소도 구경하고, git 관리법이라던지 repo 관리하는 걸 훔쳐배우고 있었어요. 이번 기회에 github 사용법을 꽤 자세하게 배워서 열심히 정리했어요!

실습 내용은 github으로 organization을 생성하여 팀원들과 어떻게 github flow로 협업을 수행할 수 있는지, github UI를 활용하여 편리하게 작업하는 법을 배웠습니다. 팀장이 해야하는 일과 프로젝트 팀원이 해야하는 일을 정리했으니 팀 프로젝트를 하게 될 때 참고하면 될거 같아요~

## 초기 세팅(팀장)

1. github 홈페이지에서 오른쪽 상단 `+` 눌러 `New organization` 생성
2. 생성 후 People에서 팀원 초대
   - 왠만하면 팀장은 한명만 하는 게 낫다.
3. 작업할 repository 생성
4. Issue 탭에서 milestone 생성
   - 단기적인 작업 목표 단위
   - 제목, 마감일, 해야할 일 목록
5. Issue 탭 label 생성
   - [Sane GitHub Labels](https://medium.com/@dave_lunny/sane-github-labels-c5d2e6004b63)
   - 중요도 Priority
   - 타입 Type
   - 상태 Status
6. Settings - Features - Issues에서 issue templates 작성
   - 나중에 팀원들이 issue를 작성할 때 기본 템플릿으로 활용 가능하다.  
     ![issue-templates](/assets/images/2023/05/02/issue-templates.png)
   - 이슈는 이렇게 출력될 것이다.  
     ![issue-templates](/assets/images/2023/05/02/issue-templates-preview.png)
   - issue templates를 만들면 repository에 .github/ISSUE_TEMPLATE/에 템플릿이 생성되어 커밋됨

## Projects

여러 이슈가 있을 때, 프로젝트 보드를 만들어 관리하는게 추천된다.

1. repo의 Projects 항목
2. `Link a project` 초록 버튼 오른쪽의 화살표로 `New projcet` 생성
3. 쉽고 단순하게 Project templates의 Team backlog로 가자!
   - sprint의 backlog는 보통 Scrum board 형식을 사용함
   - `New`, `Backlog`, `Ready`, `In progress`, `In review`, `Done`
   - 모든 Issue는 `New`에 존재
   - 이번 Sprint에서 작업할 Issues는 `Backlog`에 둔다.
   - 과정을 거쳐 기간안에 전부 `Backlog`에서 `Done` 옮기는게 목표
4. `+ Add item` - `+` - `Add from repository`로 issue들 scrum board에 추가해두자.

> 이런 모든 관리는 팀장이 제어한다.

## 작업(팀원)

1. 할일을 먼저 Issue로 등록(템플릿 사용)
2. `repo`를 자신의 계정으로 `Fork`
3. 자신 `Fork`한 걸 local로 `clone`
4. 작업할 issue 기능에 맞게 `features/제작_기능`으로 `branch`생성
5. Issue의 task 리스트에 따라 작업 단위를 나누어 커밋 수행
   - 이때 task 하나 완료할 때 마다 github의 `Issue`에서 체크리스트를 체크해준다.
6. `Fork`한 자신의 원격 저장소로 `git push -u origin feature/제작_기능` push 수행
7. 저장소를 보면 `Compare & pull request` 있는데 팀 저장소로 `pull request` 수행

   - 여러군데 pull request 버튼 있는데 어딜 눌러도 같은 화면으로 감
   - 팀 저장소의 `main` <- 내 저장소의 `feature/제작_기능`으로 보내야함
   - Title 방치하지말고 무슨 작업했는지 알아보기 쉽게 작성
   - 내용 comment에도 작업 내용 상세히 작성
   - 예시

     ```md
     ## 설명

     resolve #1

     사용자가 fizzbuzz()를 수행할 때 전달인자로 숫자를 입력하여 실행하면
     1부터 해당 수까지 1씩 증가시키면서,
     3의 배수라면 'fizz',
     5의 배수라면 'buzz',
     15의 배수라면 'fizzbuzz'

     ## To reviewers,

     - fizzbuzz()시 수를 입력하여 수행하세요.
     ```

     - comment는 마크다운 양식을 따른다!
     - `resolve #1`은 Issue #1번을 자동으로 close 시키도록 연결시키는 역할
     - pull request가 merge되면 Issue 또한 자동으로 close된다.
     - `close`, `closed` `fix`, `fixed`, `resolve` 등 가능

8. pull reqeust가 `merge`되는 등 작업할 필요가 없어지면 작업한 `branch`를 꼭 삭제해주자.

## 프로젝트 진행 도중 팀장이 계속 해야 하는 것

- Issues
  - Assignee(작업자) 부여
  - 어떤 MileStone인지 부여
    - 이를 통해 언제 처리해야할지 판단할 수 있다.
  - 프로젝트 달고 관리
- Pull requests
  - Reviewers 부여
    - 코드 작성자 외에 다른 사람으로
  - Assignees(작성자) 부여
  - Label도 달기

## 코드 리뷰

1. Pull requests들어가서 기본 comments와 commits들 확인
2. File changed 항목으로 가서 수정한 코드 확인
   - 원하는 줄에 comment 달 수 있다.
   - 코드 위에 필요한 comment를 달고 나면 `Finish your review`(초록색 버튼)으로 마치기
     - Comment : 승인과 무관한 일반적인 커멘트
     - Approve : Comment와 다르게 리뷰어가 승인을 하는 것(머지해도 괜찮다!)
     - Request changes : 변경 사항 요청, 승인을 거부

> 승인이나 승인 거부 시, Pull request창에 승인 여부가 나타난다.  
> 작업자는 승인 거부되면 다시 작업을 해야한다.

## Pull request가 Request changes되었을 때

1. 작업하던 `Fork`뜬 repo의 `feature/제작_기능` branch에서 계속 작업한다.
2. 요청사항들에 맞게 commit들 생성
3. commit들 다 만들었으면 내 저장소로 `push`
4. 자동으로 `pull request`에도 반영이 된다.
   - 리뷰어가 다시 리뷰수행하고 승인/수정 요청을 한다.
   - 승인되었으면 `Merge pull request`하면 됨

## 팀원이 `Fork`하여 작업할 때

팀 저장소의 상태로 저장소를 계속 업데이트해야한다.

- `git remote -v` 시 origin으로 본인 저장소만 나온다.
- `git remote add upstream https://github.com/팀/팀_repo.git`
- 이제 원격 저장소가 2개가 달려있다.
  - push할 때, `git push -u origin feature/기능`
  - pull할 때, `git pull upstream main`
  - 나눠서해야한다.

---

## ✏️여담

이론 파트 동영상을 보고 혼자 컴퓨터랑 노트북으로 팀장/팀원으로 나눠서 열심히 실습해봤어요.👨‍💻👨‍💻

github 기능들 예전부터 자세히 알고 싶었는데 찾아봐서 어려워서 미루다가 제대로 배워서 좋아요! 졸업과제 할때 팀장으로 잘 적용해봐야겠어요.

팀원들끼리 한 프로젝트 할 때는 굳이 `Fork`하지 않고 팀 repository에서 `branch` 만들어서 다 같이 바로 작업하는 것도 한 가지 방법이 될 거 같아요.

카테캠 듣는 보람이 있네요. 재밌어요!
