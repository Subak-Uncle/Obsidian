# labels.json

~~~json
[
{
    "name": "🥰 Accessibility",
    "color": "facfcf",
    "description": "웹접근성 관련"
},
{
    "name": "✨ Feature",
    "color": "a2eeef",
    "description": "기능 개발"
},
{
    "name": "🌏 Deploy",
    "color": "C2E0C6",
    "description": "배포 관련"
},
{
    "name": "🗨️ Comment",
    "color": "6B60FB",
    "description": "필요한 주석 추가 및 변경"
},
{
    "name": "🐞 BugFix",
    "color": "d73a4a",
    "description": "버그 수정"
},
{
    "name": "⚒️Fix",
    "color": "446265",
    "description": "피드백 사항 수정"
},
{
    "name": "💻 CrossBrowsing",
    "color": "C5DEF5",
    "description": "브라우저 호환성"
},
{
    "name": "📃 Docs",
    "color": "1D76DB",
    "description": "문서 작성 및 수정 (README.md 등)"
},
{
    "name": "📬 API",
    "color": "D4C5F9",
    "description": "서버 API 통신"
},
{
    "name": "🔨 Refactor",
    "color": "f29a4e",
    "description": "코드 리팩토링"
},
{
    "name": "🎯  Test",
    "color": "ccffc4",
    "description": "test 관련(storybook, jest...)"
},
{
    "name": "⚙ Setting",
    "color": "e3dede",
    "description": "개발 환경 세팅"
},
{
    "name": "🙏 Request",
    "color": "bfdadc",
    "description": "기능 요청"
}
]
~~~





# 맥북 vers

## 준비물

- json 파일
- access 토큰
- node.js 설치

터미널 창에서

~~~cmd
>> cd {labels.json 파일 위치}
~~~

~~~cmd
npx github-label-sync --access-token {본인의 token} --labels ./labels.json {라벨을 업데이트하고 싶은 본인의 레포지터리}
(주의 : 레포지토리 주소 뒤에 .git을 제거해주어야 함.
ex) hot-gamza/loveloveshot-spring-boot)
~~~

을 누르면 

~~~cmd
npx i github-label-sync
~~~

깃허브 파일 설치할 것이냐고 물어본다. y를 누르면 적용되는 것을 확인할 수 있다!!

