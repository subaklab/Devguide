# Contributing

핵심 개발팀과 커뮤니티의 연락처 정보를 아래에서 찾을 수 있습니다. PX4 프로젝트는 3개 브랜치 Git이 있는데 :

  * [master](https://github.com/px4/firmware/tree/master) 기본으로 안정버전이 이니며 빠르게 개발 중인 버전
  * [beta](https://github.com/px4/firmware/tree/beta) 철저히 테스트하고 있는 버전으로 비행 테스터를 위한 버전
  * [stable](https://github.com/px4/firmware/tree/stable) 최신 릴리즈 버전

[rebase를 통한 진화과정](https://www.atlassian.com/git/tutorials/rewriting-history)을 보고 [Github flow](https://guides.github.com/introduction/flow/)을 피하고자 합니다. 그러나 글로벌 팀과 빠른 개발로 가끔은 merge하기 위해 다시 정렬해야하는 경우도 있습니다.

새로운 기능 개발에 기여하고자 한다면, [Github 계정만들고](https://help.github.com/articles/signing-up-for-a-new-github-account/) 저장소를 [fork](https://help.github.com/articles/fork-a-repo/)하고 [새로운 branch 만들고](https://help.github.com/articles/creating-and-deleting-branches-within-your-repository/) 변경 내용을 추가하고 마지막으로 [pull request 보내기](https://help.github.com/articles/using-pull-requests/)를 합니다. 변경 내용이 [continuous integration](https://en.wikipedia.org/wiki/Continuous_integration)을 통과하면 merge 됩니다.

모든 contribution은 [BSD 3-clause 라이센스](https://opensource.org/licenses/BSD-3-Clause)를 따르며 모든 코드는 반드시 사용에 제약이 없어야 합니다.

## 커밋과 커밋 메시지 (Commits and Commit Messages)

모든 변경에 대한 커밋 메시지는 커밋 내용을 설명하고 상세하게 기록하도록 합니다. 잘 구조화하면 한줄로 요약할 수 있겠지만 상세하게 설명을 제공하도록 합니다.

```
Component: Explain the change in one sentence. Fixes #1234

Prepend the software component to the start of the summary
line, either by the module name or a description of it.
(e.g. "mc_att_ctrl" or "multicopter attitude controller").

If the issue number is appended as <Fixes #1234>, Github
will automatically close the issue when the commit is
merged to the master branch.

The body of the message can contain several paragraphs.
Describe in detail what you changed. Link issues and flight
logs either related to this fix or to the testing results
of this commit.

Describe the change and why you changed it, avoid to
paraphrase the code change (Good: "Adds an additional
safety check for vehicles with low quality GPS reception".
Bad: "Add gps_reception_check() function").

Reported-by: Name <email@px4.io>
```

**```git commit -s```를 사용하면 모든 커밋에 사인을 남길 수 있습니다.** 마지막 라인에 ```signed-off-by:```과 이름과 이메일을 추가합니다.

커밋 가이드는 Linux Kernel과 리누스 토발즈가 [관리하는 프로젝트](https://github.com/torvalds/subsurface/blob/a48494d2fbed58c751e9b7e8fbff88582f9b2d02/README#L88-L115) 의 좋은 예들을 기반으로 하고 있습니다.

## 비행 결과 테스트 (Tests Flight Results)

비행 테스트는 QA 관점에서 중요합니다. microSD 카드에서 log 파일을  [Log Muncher](http://logs.uaventure.com)에 업로드하고 비행 결과와 링크를 [PX4 Discuss](http://discuss.px4.io/)에 공유해 주세요.

## 포럼과 챗팅 (Forums and Chat)

  * [Google+](https://plus.google.com/117509651030855307398)
  * [Gitter](https://gitter.im/PX4/Firmware)
  * [PX4 Discuss](http://discuss.px4.io/)

## 주단위 개발 콜 (Weekly Dev Call)

PX4 개발 팀은 매주 개발관련 내용에 대해서 대화를 나눕니다. ([Mumble](http://mumble.info) 클라이언트로 연결)
The PX4 Dev Team syncs up on its weekly dev call (connect via [Mumble](http://mumble.info) client).

  * TIME: 19:00h Zurich time, 1 p.m. Eastern Time, 10 a.m. Pacific Standard Time
  * Server: sitl01.dronetest.io
  * Port: 64738
  * Password: px4
  * The agenda is announced in advance on the [PX4 Discuss](http://discuss.px4.io/c/weekly-dev-call)
  * Issues and PRs may be labelled "devcall" to flag them for discussion
