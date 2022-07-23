프로젝트 진행 도중 s3 cloud를 사용하기 위해 `spring-cloud-stater-aws`의존성을 추가하였는 데 

`com.amazonaws.SdkClientException: Failed to connect to service endpoint`와 같은 예외가 발생하였다. (실행에는 문제가 없었다.)

찾아보니 로컬환경은 aws환경이 아니라서 나는 예외였다.  즉, aws 환경에서는 문제가 없다는 것이다. 

문제점이 하나가 더 있었는데 로컬,테스트에서 실행 시 잠시 멈췄다가 실행되는 문제 발생하였다. 

몇초 걸리지 않지만 빠르게 테스트, 빌드, 실행하기 위해선 어떻게 해야 할까? 



## 해결방안 

- 로컬

	- vm option에 `-Dcom.amazonaws.sdk.disableEc2Metadata=true` 추가

- 테스트

	```java
	static {
	    System.setProperty("com.amazonaws.sdk.disableEc2Metadata", "true");
	}
	```

	- 테스트의 경우 `ApiTest`라는 추상 클래스를 만든 후 위의 코드를 추가하고 `Controller`테스트시 상속하여 해결하였다.