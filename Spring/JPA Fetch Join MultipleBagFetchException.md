# JPA Fetch Join MultipleBagFetchException

이번 글에서는 프로젝트 중 발생한  MultipleBagFetchException에 대해 정리해보고자 한다. 


## MultipleBagFetchException
![](https://i.imgur.com/8n2tkOx.png)
프로젝트에서는 게시글이 위와 같은 모습으로 관계를 가지고 있다.  이에 게시글을 조회 시 post_image와 post_tag를 모두 fetch join하여 가져오면 한 방 쿼리가 되겠지~? 라며 fetch join을 한 후 테스트를 돌려보았는 데 `MultipleBagFetchException` 에러가 발생하였다.
![](https://i.imgur.com/OVAsq45.png)

![](https://i.imgur.com/B3oROEp.png)

왜 이러한 문제가 생겼나 찾아보니 fetch join하는 대상에 있었다. 
먼저 여기서 말하는 Bag이란 무엇일까? 
> A `<bag>` is an unordered collection, which can contain duplicated elements. That means if you persist a bag with some order of elements, you cannot expect the same order retains when the collection is retrieved. There is not a “bag” concept in Java collections framework, so we just use a `java.util.List` corresponds to a `<bag>`.
> https://stackoverflow.com/questions/13812283/difference-between-set-and-bag-in-hibernate

즉, Bag은 Set과 같이 순서가 없고, List와 같이 중복을 허용하는 자료구조이다. 하지만 자바 컬렉션 프레임워크에서는 Bag이라는 개념이 없기 때문에 List를 Bag으로써 사용하고 있는 것이다. 
현재 post_image, post_tag는 모두 List로 매핑하고 있다.
![](https://i.imgur.com/cvr75kf.png)
![](https://i.imgur.com/fvdtzAa.png)

그럼 이것이 왜 문제일까? 
fetch join을 복수의 컬렉션에 적용하게 되면 카테시안 곱에 의해 중복 데이터가 발생하기 때문에 미리 `MultipleBagFetchException`를 띄우며 에러처리를 하는 것이다. 

> 카테시안 곱 
> 발생가능한 모든 경우의 수의 행이 조회되는 것 
> 즉, N 개의 행을 갖진 테이블과 M개의 행을 가진 테이블의 카타시안 곱은 N * M이 되는 것

즉, 정리하자면 JPA에서 Fetch Join은 아래와 같은 특징을 가지고 있다. 
- ToOne 관계에서는 몇개든 사용이 가능하다. 
- ToMany 관계에서는 1개만 사용이 가능하다.



### 해결법 

그렇다면 어떻게 해결해야할까? 아래와 같은 해결방법이 있다. 
- list 대신 set을 사용해서 중복을 제거한다. 
- 모든 자식 테이블을 다 Lazy Loading으로 ( N + 1 문제 발생 )
- 가장 데이터가 많은 자식 쪽에 Fetch Join을 걸고 나머지는 모두 Lazy Loading 
	- `hibernate.default_batch_fetch_size` 적용으로 `in` 쿼리로 성능 보장
- Fetch Join을 나누어서 실행한 후에 조합하기 

이에 나는 post_tag의 경우 중복을 허용하지 않으니 첫번째 방법처럼 list -> set으로 바꿔주는 것을 선택하였다. 
![](https://i.imgur.com/wnnO7F0.png)

이것으로 문제는 해결된 지 알고  행복하게 배포를 진행하였다. 하지만 .....



## 이미지 중복 문제점

행복하게 배포한 뒤 다른 작업을 진행하고 있는 데 프론트 팀원분이 이렇게 말씀해주셨다. 
> 이미지가 중복해서 내려오는 것 같아요! 하나만 업로드 했는 데도 똑같은 게 두 개가 내려와요!!

이에 나는 바로 어디가 문제인지 확인하게되었고 게시물의 Fetch Join부분이 잘못되었다는 것을 알게 되었다.

```sql
select
    post0_.id as id1_3_0_,
    postimages1_.id as id1_4_1_,
    posttags2_.id as id1_5_2_,
    tag3_.id as id1_6_3_,
    couple4_.id as id1_0_4_,
    post0_.created_date_time as created_2_3_0_,
    post0_.updated_date_time as updated_3_3_0_,
    post0_.content as content4_3_0_,
    post0_.couple_id as couple_i9_3_0_,
    post0_.dating_date as dating_d5_3_0_,
    post0_.latitude as latitude6_3_0_,
    post0_.longitude as longitud7_3_0_,
    post0_.title as title8_3_0_,
    postimages1_.image_url as image_ur2_4_1_,
    postimages1_.post_id as post_id3_4_1_,
    postimages1_.post_id as post_id3_4_0__,
    postimages1_.id as id1_4_0__,
    posttags2_.post_id as post_id2_5_2_,
    posttags2_.tag_id as tag_id3_5_2_,
    posttags2_.post_id as post_id2_5_1__,
    posttags2_.id as id1_5_1__,
    tag3_.color as color2_6_3_,
    tag3_.couple_id as couple_i4_6_3_,
    tag3_.name as name3_6_3_,
    couple4_.created_date_time as created_2_0_4_,
    couple4_.updated_date_time as updated_3_0_4_,
    couple4_.start_date as start_da4_0_4_
from
    post post0_
        inner join
    post_image postimages1_
    on post0_.id=postimages1_.post_id
        inner join
    post_tag posttags2_
    on post0_.id=posttags2_.post_id
        inner join
    tag tag3_
    on posttags2_.tag_id=tag3_.id
        inner join
    couple couple4_
    on post0_.couple_id=couple4_.id
where
        post0_.id=?
```

![Untitled](https://tva1.sinaimg.cn/large/e6c9d24egy1h5ddiq5yu9j20hz05cwf1.jpg)

SQL문으로 확인한 결과 당연하게 이미지와 태그가 중복되어서 조회되게 된다.

![](https://i.imgur.com/Fnnk50t.png)

하지만 어플리케이션 단에서 태그는 중복 제거가 되지만 이미지는 중복 제거가 되지 않고 그대로 조회가 된다. 



### 뭐가 문제지?

문제점은 무엇일까? 사실 해당 문제점은 위에서 설명한 부분에서 찾을 수 있다. 

post_image는 **중복을 허용하는 list**이고 post_tag는 **중복을 허용하지 않는 set**이기 때문이다. 

이를 모르고 처음에는 distinct를 해주지 않아서 그런가 싶었지만 소용이 없었고 본질적인 문제였던 것이다.



### distinct는 왜 안될까? 

JPA에서 distinct는 다음과 같은 효과가 있다.

- SQL의 distinct는 중복된 결과를 제거하는 명령 
- JPQL의 distinct는 2가지 기능을 한다. 
	- SQL에 distinct를 추가 
	- 어플리케이션에서 같은 식별자를 가진 엔티티 중복 제거 

distinct는 select 대상 ( 여기서는 post )에 대해서 중복제거를 하는 것이지 fetch join하는 대상(post_image)에 대해 하는 것이 아니기 때문에 소용이 없던 것이다. ( 이미 post는 하나 일 뿐 이기 때문에 )



여기서 개인적인 궁금점으로 SQL에선 게시물 중복이 발생하는 데 `distinct`을 사용하지 않아도 어떻게 중복을 해결해주는 것일까?

아마 추측을 해보자면 반환값이 `Post`이기 때문에 `distinct`가 없어도 자동으로 중복 제거하여 반환해주는 것 같다. ( 좀 더 알아보고 수정할 예정! )

만약 List로 반환한다면 아래와 같이 post가 중복되어서 나오게 된다.

![Untitled](https://tva1.sinaimg.cn/large/e6c9d24egy1h5ddj2z79hj20hi0b7wg4.jpg)

여기서 `distinct` 를 적용 시 중복이 제거된다.

![Untitled](https://tva1.sinaimg.cn/large/e6c9d24egy1h5ddj5qt9jj20hh0a375q.jpg)



### 그럼 이미지도 Set으로 변경한다면?

![Untitled](https://tva1.sinaimg.cn/large/e6c9d24egy1h5ddislhufj20h00cvq4r.jpg)

중복없이 둘 다 가져온다는 것을 볼 수 있다. 그럼 해결이 된 것일까? 



### 사실 이미지는....

하지만 이미지는 썸네일 이미지를 조회할 수 있어야 하기 때문에 순서가 중요하다.

![Untitled](https://tva1.sinaimg.cn/large/e6c9d24egy1h5ddiux3r9j20gi04rq33.jpg)

그렇기 때문에 결국 아래와 같은 방법으로 해결하였다.


![](https://i.imgur.com/lxLOAoK.png)

```sql
select
        post0_.id as id1_3_0_,
        posttags1_.id as id1_5_1_,
        tag2_.id as id1_6_2_,
        couple3_.id as id1_0_3_,
        post0_.created_date_time as created_2_3_0_,
        post0_.updated_date_time as updated_3_3_0_,
        post0_.content as content4_3_0_,
        post0_.couple_id as couple_i9_3_0_,
        post0_.dating_date as dating_d5_3_0_,
        post0_.latitude as latitude6_3_0_,
        post0_.longitude as longitud7_3_0_,
        post0_.title as title8_3_0_,
        posttags1_.post_id as post_id2_5_1_,
        posttags1_.tag_id as tag_id3_5_1_,
        posttags1_.post_id as post_id2_5_0__,
        posttags1_.id as id1_5_0__,
        tag2_.color as color2_6_2_,
        tag2_.couple_id as couple_i4_6_2_,
        tag2_.name as name3_6_2_,
        couple3_.created_date_time as created_2_0_3_,
        couple3_.updated_date_time as updated_3_0_3_,
        couple3_.start_date as start_da4_0_3_ 
    from
        post post0_ 
    inner join
        post_tag posttags1_ 
            on post0_.id=posttags1_.post_id 
    inner join
        tag tag2_ 
            on posttags1_.tag_id=tag2_.id 
    inner join
        couple couple3_ 
            on post0_.couple_id=couple3_.id 
    where
        post0_.id=?
```

```sql
select
        postimages0_.post_id as post_id3_4_1_,
        postimages0_.id as id1_4_1_,
        postimages0_.id as id1_4_0_,
        postimages0_.image_url as image_ur2_4_0_,
        postimages0_.post_id as post_id3_4_0_ 
    from
        post_image postimages0_ 
    where
        postimages0_.post_id=?
```

![Untitled](https://tva1.sinaimg.cn/large/e6c9d24egy1h5ddiy8uvjj20gy0cv0ul.jpg)

![Untitled](https://tva1.sinaimg.cn/large/e6c9d24egy1h5ddj11bzkj20e503pq38.jpg)

post_tag의 경우 많은 데이터를 가질 수 있으니 fetch join으로 적용해주고 post_image는 현재 최대 5가지로 정책을 정했기 때문에 DTO로 변환 시 조회하는 것으로 해결하였다. 즉, 위에서 말한 해결법 중 3번째 방법을 사용한 것이다. 

처음에는 `Hibernate default_batch_fetch_size`도 사용해야지 했는 데 생각해보니 post가 하나이기 때문에 in절을 사용하는 것이 아닌 =절을 사용하기 때문에 필요없는 옵션이였다. ( 다중 조회에선 필요하겠지..? )



## 결론 

역시 어떤 기술이라도 기본이 중요하다는 것을 뼈저리게 느끼게 되었다. 

JPA를 사용하면서 컬렉션이면 `그냥 fetch join해야지~`라며 아무 생각없이 사용했던 것 같다. 

또한, 제대로 알아보지도 않고 그냥 해결했다고 끝내버리니 이러한 문제가 발생했다. 

이러한 문제가 발생하기 이전에 해당 기술의 기본을 정확히 알고 해결하는 습관을 들여야 겠다고 생각했고 다시 한번 `왜?라는 생각을 하며 구현`해야겠다고 느끼게 되었다.

마지막으로, 쓸모 있는 테스트의 중요성을 느끼게 되었다. 사실 이미지 중복 문제는 제대로 테스트를 짰다면 배포전에 잡을 수 있던 버그였다. 하지만 그냥 null이 아닌지만 확인하였고 쓸모 없는 테스트를  진행함으로써 이러한 문제가 발생한 것이였다. 좀 더 테스트 코드를 짤 때 꼼꼼히 짜야 겠다는 생각을 하였다.

만약 다음에도 이러한 문제가 발생한다면 내가 처음에 선택한 자료형을 바꾸는 방법이 아닌 가장 데이터가 많은 자식쪽에 `fetch join`을 사용하고 나머지 부분은 `Hibernate default_batch_fetch_size` 적용으로 성능을 보장하는 방향으로 해결해 나갈 것이다.



## Reference 

- [dev-tips/JPA - Hibernate MultipleBagFetchException.md at master · HomoEfficio/dev-tips](https://github.com/HomoEfficio/dev-tips/blob/master/JPA%20-%20Hibernate%20MultipleBagFetchException.md)

- [(Troubleshooting) Hibernate MultipleBagFetchException 정복하기](https://perfectacle.github.io/2019/05/01/hibernate-multiple-bag-fetch-exception/)

- [MultipleBagFetchException 발생시 해결 방법](https://jojoldu.tistory.com/457?fbclid=IwAR132BRMYHrL4D5Pu25YUglIcEN1FGTE2tacFcsVOPAT0MAzwoMX6Flzbe0)

- [[JPA] Fetch Join 할 때 MultipleBagFetchException 해결하는 법](https://devlog-wjdrbs96.tistory.com/421)

- [JPA 에서 별칭을 쓰지 않는 이유 (하지만 쓴 이유)](https://yjksw.github.io/jpa-fetch-join-nickname/)

- [JPA의 사실과 오해 (사실은 JPQL과 Fetch Join의 오해)](https://os94.tistory.com/201)

- [A Guide to MultipleBagFetchException in Hibernate | Baeldung](https://www.baeldung.com/java-hibernate-multiplebagfetchexception)
