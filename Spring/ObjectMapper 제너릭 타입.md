현재 프로젝트에서 MockMvc를 통해 테스트를 작성하고 있다.

그렇기 때문에 body에 json타입을 넣으려면 ObjectMapper를 통해 기입해줘야한다.  (`writeValueAsString`)

여기까지는 문제가 없지만 검증할 때 문제가 생기게 된다.

나는 mockMvc.andReturn().getResponse()를 통해 `MockHttpServletResponse`를 가져와서 검증하는 것을 선호한다. 이렇게 하면 좀 더 검증부분가 행동부분이 구분이 가서 더 깔끔해지기 때문이다.

하지만 현재 프로젝트는 ApiResponse<T>라는 제너릭 객체를 통해 결과값을 반환해주고 있다.

```java
@Getter
@NoArgsConstructor(access = AccessLevel.PRIVATE)
public class ApiResponse<T> {

    private T data;

    public ApiResponse(T data) {
        this.data = data;
    }
}
```

이럴 경우 아래처럼 ObjectMapper의 `readValue`로 바로 객체를 반환받을 수 없다.

```java
objectMapper.readValue(response.getContentAsString(), ApiRespone<T>.class) 
// 불가능
```

제너릭은 컴파일타임에서만 타입 파라미터를 확인하고, 이후에는 원래 클래스 타입만 남는 Type Erasure 특징을 가지고 있기 때문에 불가능한 것 이다.

이에 해결법을 찾고자 서칭을 하게 되었고 나와 같은 고민을 한 사람들이 쓴 글을 보게 되었다.

`objectMapper.getTypeFactory().constructParametricType()`를 통해 JavaType를 추출하여 형변환을 할 수 있음을 알게 되었다. 추출한 JavaType을 통해 객체로 반환받을 수 있는 것이다.

```java
JavaType javaType = objectMapper.getTypeFactory().constructParametricType(ApiResponse.class, TokenResponse.class);
```

해당 JavaType을 `readValue`를 쓰던 것처럼 사용해주면 된다.

프로젝트에서 자주 사용될 것 같아 메서드로 만들어 안의 data 객체를 가져올 수 있도록 만들었다.

```java
protected <T> T getResponseObject(MockHttpServletResponse response, Class<T> type) throws IOException {
    JavaType javaType = objectMapper.getTypeFactory().constructParametricType(ApiResponse.class, type);
    ApiResponse<T> result = objectMapper.readValue(response.getContentAsString(), javaType);
    return result.getData();
}
```

저번 프로젝트에서도 해당 문제점이 발생하여 `TypeReference`를 사용하여 해결하였지만 이는 메서드로 추출에 실패하여 `jsonPath`로 검증했었다.

하지만 사용하면서도 찜찜했던 것이 행동과 검증의 경계점이 모호해진다고 나는 생각이 들었다.

그렇기 때문에 위에서 해결한 방식으로 나는 검증을 해나갈 것 같다.



## Reference

- [Generic Json Parsing with ObjectMapper](https://jinwooe.wordpress.com/2015/11/24/generic-json-parsing-with-objectmapper/)

- [java - Jackson and generic type reference](https://technoteshelp.com/java-jackson-and-generic-type-reference/)