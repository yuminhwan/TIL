## permitAll()

- `permitAll`을 했다고 해서 필터가 무시되는 것이 아니다!

- **필터를 무시할려면 직접 필터안에서 구현해줘야 한다!**

	- `shouldNotFilter` 사용

	```java
	@Override
	protected boolean shouldNotFilter(HttpServletRequest request) {
	    return request.getRequestURI().endsWith("tokens") && request.getMethod().equalsIgnoreCase("POST");
	}
	```



## 로그인한 유저의 id 가져오기 (컨트롤러에서)

```java
@PostMapping
public ResponseEntity<ReservationResponseForGuest> createReservation(
    @Valid @RequestBody CreateReservationRequest request,
    @AuthenticationPrincipal JwtAuthentication user
) {
    return ResponseEntity.ok(reservationService.createReservation(user.id(), request));
}
```

- `@AuthenticationPrincipal JwtAuthentication user` 로 인증객체를 가져온다
- `record`라면 `user.id()` 로 아이디 값을 가져온다!