## JPA exists

```java
boolean existsByName(String name);
```

Spring Data Jpa 에선 메서드로 쿼리를 만들어서 사용한다.

하지만 조금이라도 복잡해진다면 메서드로 표현이 힘들때가 있다.

![스크린샷 2022-07-11 오후 3.36.22](https://tva1.sinaimg.cn/large/e6c9d24egy1h4jk23fx38j212809iq49.jpg)

![스크린샷 2022-07-11 오후 3.36.11](https://tva1.sinaimg.cn/large/e6c9d24egy1h4jk29xj1uj20ne0f2js0.jpg)

- 예약하고 싶은 기간에 이미 숙소가 예약중인지 확인하는 쿼리
	- 메서드로만 표현하기엔 너무 길어지고 복잡하다.

이런 경우 `@Query` 를 사용하지만 JPQL에선 select의 exists 를 지원하지 않는다.

하지만 where절에서는 가능하기에 count쿼리로 우회하여 사용한다.

![스크린샷 2022-07-11 오후 3.43.42](https://tva1.sinaimg.cn/large/e6c9d24egy1h4jk2er08gj20w00dwgnk.jpg)

![스크린샷 2022-07-11 오후 3.43.33.png](https://tva1.sinaimg.cn/large/e6c9d24egy1h4jk2ieu2mj20ny0eojs8.jpg)

- 하지만 이 방식은 성능상 문제가 있다.

## count VS exists

- `exists`의 경우 첫번째 결과에서 바로 true를 리턴하여 빠르다.
- 하지만 `count`의 경우 결국 총 몇건인지 확인해야하기 때문에 전체를 확인할 수 밖에 없어 `exists`보다 느리다.

## QueryDsl에서의 exists

- 블로그에선 QueryDsl이 기본적으로 count 쿼리 방식을 사용했다고 하였지만 버전이 업그레이드됨에 따라 limit 쿼리 방식으로 변경된 것 같다.
- `QuerydslJpaPredicateExecutor.exists`
- 사용해보려 했지만 실패…. 후에 공부한 뒤 다시 봐야겠다.

![스크린샷 2022-07-11 오후 4.07.38](https://tva1.sinaimg.cn/large/e6c9d24egy1h4jk2oflxaj20zy070aar.jpg)

- `limit 1`를 통해 1개만 조회하여 있는 지 없는 지 판단

```java
@Override
public boolean existReservationByRoom(Room room, Long reservationId, ReservationDate reservationDate) {
    LocalDate checkIn = reservationDate.getCheckIn();
    LocalDate checkOut = reservationDate.getCheckOut();

    Integer fetchOne = queryFactory.selectOne()
        .from(reservation)
        .where(
            eqRoom(room)
            , notEqReservationId(reservationId)
            , notInCanceled() // 상태가 취소가 아닌 것
            , betweenCheckIn(checkIn, checkOut).or(betweenCheckOut(checkIn, checkOut))
        )
        .fetchFirst();  // limit 1

    return fetchOne != null;
}
```

![스크린샷 2022-07-11 오후 4.09.39](https://tva1.sinaimg.cn/large/e6c9d24egy1h4jk2v3ec1j20nu0kqt9n.jpg)

## 결론

- 오버엔지니어링일 수 도 있겠지만 좋은 방식이 있는 데 그것을 따라가지 않을 이유가 없어서 구현해보았다.
- 데이터가 많지 않아 크게 체감되지 않지만 좀 더 안정적인 프로그램이 되지 않았나 생각한다!

## Reference

- [JPA exists 쿼리 성능 개선](https://jojoldu.tistory.com/516)
- [[Querydsl\] 다이나믹 쿼리 사용하기](https://jojoldu.tistory.com/394)