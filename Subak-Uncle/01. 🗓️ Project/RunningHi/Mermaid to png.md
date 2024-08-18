
클로드를 통해 `Sequence Diagram` 을 완성하였습니다.
클로드 측에서 설명을 토대로 `.mermaid` 파일을 만들어주는데요!!
`cli` 기반으로 `Sequence Diagram` 을 만들어주는 방식입니다.

https://mermaid.live/ 에 이동하여 파일을 '드래그 앤 드롭'을 해주어 이미지 형태로 볼 수 있습니다만!!

더 편한 방법이 있습니다.

터미널에서 node를 활용해 라이브러리를 설치해줍니다.

```bash
1. 라이브러리 설치
npm install -g @mermaid-js/mermaid-cli

2. 버전 확인
npx mmdc --version

3. 이미지 변환 명령어
npx mmdc -i {mermaid 파일 위치} -o {이미지 파일 명.확장자}
```

결과물은 다음과 같습니다.

```bash
sequenceDiagram
    participant User
    participant iOS App
    participant Spring Server
    participant FCM Server
    participant DB

    User->>iOS App: 로그인
    iOS App->>iOS App: 알림 동의 여부 확인
    alt 알림 동의
        iOS App->>FCM Server: FCM 토큰 요청
        FCM Server-->>iOS App: FCM 토큰 발급
        iOS App->>Spring Server: FCM 토큰 저장 요청
        Spring Server->>DB: FCM 토큰 저장
        DB-->>Spring Server: 저장 완료
        Spring Server-->>iOS App: 저장 완료 응답
    end

    Note over Spring Server: 알림 트리거 이벤트 발생<br>(댓글, 챌린지, 레벨업, 문의답변 등)

    Spring Server->>DB: 알림 엔티티 저장
    DB-->>Spring Server: 저장 완료

    Spring Server->>FCM Server: 푸시 알림 발송 요청
    FCM Server->>iOS App: 푸시 알림 전송

    alt 알림 동의한 경우
        iOS App->>User: 푸시 알림 표시
    end

    User->>iOS App: 앱 실행
    iOS App->>Spring Server: 알림 리스트 요청
    Spring Server->>DB: 알림 데이터 조회
    DB-->>Spring Server: 알림 데이터 반환
    Spring Server-->>iOS App: 알림 리스트 JSON 응답

    alt 사용자가 알림 선택
        User->>iOS App: 알림 선택
        iOS App->>iOS App: targetPage와 targetId로 페이지 이동
        iOS App->>Spring Server: 알림 읽음 처리 요청
        Spring Server->>DB: 알림 읽음 상태 업데이트
        DB-->>Spring Server: 업데이트 완료
        Spring Server-->>iOS App: 읽음 처리 완료 응답
    else 사용자가 알림 닫기 ('x' 표시)
        User->>iOS App: 알림 닫기
        iOS App->>Spring Server: 알림 읽음 처리 요청
        Spring Server->>DB: 알림 읽음 상태 업데이트
        DB-->>Spring Server: 업데이트 완료
        Spring Server-->>iOS App: 읽음 처리 완료 응답
    end

    Note over Spring Server: 주기적으로 실행
    Spring Server->>DB: 30일 이상 지난 읽은 알림 조회
    DB-->>Spring Server: 해당 알림 목록 반환
    Spring Server->>DB: 해당 알림 삭제
    DB-->>Spring Server: 삭제 완료
```

![[fcm.png]]