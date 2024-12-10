
## 📌 JPA Repository 예외 상황
테스트 코드를 작성하던 중,, 예상치 못한 에러를 발견하고 수정한 내용을 정리해보겠습니다.

예상치 못한 상황은 발생은 Service 부분에서 JPA를 이용해 엔티티를 조회해오는 과정에서 발견이 되었습니다.
저의 개발 의도는 "**repository를 이용해 엔티티 리스트를 조회해오자! 만약 결과가 없다면 IllegalArgumentException로 예외처리를 진행하자."**였습니다. 하지만 테스트 진행 결과가 의도치 않게 "빈 객체"가 반환되는 것을 발견한 내용입니다.

### 🎈 원인
#### service
```java
    @Transactional(readOnly = true)
    public List<GetReplyListResponse> getReplyList(Long postNo) {

        List<Reply> replyList =  replyRepository.findAllByPost_PostNo(postNo)
                .orElseThrow( () -> new IllegalArgumentException("검색 결과가 없습니다."));

        return replyList.stream()
                .filter(reply -> !reply.isDeleted())
                .map(GetReplyListResponse::fromEntity)
                .toList();
    }
```

#### repository
```java
 Optional<List<Reply>> findAllByPost_PostNo (Long postNo);
```

로 설계되어 있었습니다. getReplyList 메소드의 예외 상황을 테스트하기 위해 다음과 같이 테스트 코드를 작성했었습니다.

```java

    @ParameterizedTest
    @DisplayName("댓글 조회 테스트 : 게시글 엔티티 조회 실패 시 예외 발생 확인")
    @NullSource
//    @ValueSource(longs = 3)
    void testGetReplyListException (Long postNo) {

//        System.out.println(replyRepository.findAllByPost_PostNo(postNo));
        // when & then
        Assertions.assertThatThrownBy(
                () -> replyService.getReplyList(postNo)
        ).isInstanceOf(IllegalArgumentException.class);
    }
    
```

예상대로면, **null** 값이나 **존재하지 않는 값**을 매개변수로 받게 되면 **IllegalArgumentException**이 발생할 것이라 예상했었습니다. 하지만, 예상과 다르게 예외가 발생하지 않아 테스트에 실패했습니다.
```
Hibernate: 
    select
        r1_0.reply_no,
        r1_0.create_date,
        r1_0.is_deleted,
        r1_0.parent_no,
        r1_0.post_no,
        r1_0.reply_content,
        r1_0.reported_count,
        r1_0.update_date,
        r1_0.member_no 
    from
        tbl_reply r1_0 
    left join
        tbl_post p1_0 
            on p1_0.post_no=r1_0.post_no 
    where
        p1_0.post_no is null
```
![](https://velog.velcdn.com/images/kko0369/post/c553766f-99b2-4088-a938-ad9694360316/image.png)

**Null** 값임에도 불구하고 쿼리가 나갑니다ㅠㅠㅠ 입력 값 검증을 안했다지만 예외는 발생해야 할텐데요..ㅠㅠ? 그래서 결과 값을 직접 출력해보았습니다.
```java
System.out.println(replyRepository.findAllByPost_PostNo(postNo));

// 결과 값
>> Optional[[]]
```

흠,, 찾아보니 JPArepository는 빈 결과를 내주는 걸로 확인됩니다. 별도의 빈 결과에 대한 예외 처리가 필요하다고 생각됩니다!! 빠르게 고쳐보겠습니다.

### 🎈 해결

#### service
우선, Repository의 Optional을 지워주고, service 계층에서 직접 빈 문자열인지 판단하도록 변경하였습니다.
![](https://velog.velcdn.com/images/kko0369/post/e8394dad-5462-40cc-934d-3d30aef09a00/image.png)

#### serviceTests
이제 테스트를 돌리고 기도만 하시면 됩니다,,!!
![](https://velog.velcdn.com/images/kko0369/post/3ed81a2f-8c9e-4d22-bdd3-f1f800624241/image.png)![](https://velog.velcdn.com/images/kko0369/post/9e2b7a6c-5719-473f-b467-a104812b1f12/image.png)

무난히 통과하는 것을 확인할 수 있었습니다.
NullSource가 있을 시에 테스트의 속도가 현저히 느려지므로 repository 계층에 변수를 전달해주기 전에 꼭 valid 체크가 필요해보입니다. controller에서 dto로 매핑 시에 valid 체크를 하도록 업데이트를 하겠습니다.


### 🎈 알게 된 점
Optional 클래스는 자바 8버전부터 지원하는 NPE(Null Point Exception)을 방지하기 위해 추가된 Wrapper 클래스입니다. 하지만, JPA에서는 findAll()의 메소드를 사용할 시 List<> 형태로 결과를 반환해주므로 결과가 없을 시 빈 List<>를 결과로 반환해줍니다. 그러므로 NPE가 애시당초 발생을 안 하게 되는거죠.

반면에 단건 조회인 find() 같은 경우에는, 결과 값을 Null로 반환해줍니다. 대신, Optional을 반환타입으로 이용한다면 Optional.empty로 값을 반환해줍니다. 이후 예외처리가 가능한 것이죠!

추가로, Optional을 사용할 때 주의할 점과 사용 가이드를 정리하고 마무리하겠습니다.
- 값이 존재하지 않다면, NPE는 피해도 NoSuchElementException이 발생하게 됩니다.
- 기본적으로 Optional은 Serializable을 지원하지 않으므로 캐시나 메세지 큐 등을 이용할 때 문제가 발생합니다.
- Optional은 단순 값을 이용할 때 보다 메모리를 약 4배 정도 사용합니다.
- 

**사용 가이드**
- Optional 변수에 Null을 할당하지 말아야 합니다.
- 값이 없을 때 Optional.orElse()/orElseGet()으로 기본 값을 반환해야 합니다.
- 단순히 값을 얻으려는 목적으로만 Optional을 사용하시면 안됩니다.
- 생성자, 수정자, 메소드 파라미터 등으로 Optional을 반환하면 안됩니다.
- Collection의 경우 Optional이 아닌 빈 Collection을 사용합니다.
- 반환 타입으로만 사용합니다.

## 📒 References
- [[JPA Null 처리 방법, List가 null이 아닌 이유]](https://duooo-story.tistory.com/48)
- [[Optional을 알맞게 사용하는 방법, 사용 가이드]](https://mangkyu.tistory.com/203)