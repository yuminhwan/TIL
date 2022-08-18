# Git submodule

- 현재 팀에는 `application-security.yaml`이라는 민감정보 파일이 있습니다.
    - OAuth와 Jwt 에 대한 키값과 여러 민감정보들이 포함되어 있습니다.
- 프로젝트 기간동안 저희는 이 파일을 복사 붙여넣기 하고 `.gitignore`로 리모트에 푸쉬되지 않도록 설정하고 CI시에 젠킨스에서 ec2서버에 올라와있는 해당 파일을 복붙한 다음 빌드하도록 설정하였습니다.
- 이런 경우 문제가 파일이 변경되었을 때 항상 ec2서버에 접속하여 변경해줘야 한다는 문제점이 있으며 팀원간에 공유도 해야한다는 점이 문제입니다.
    - 만약 서버가 여러대를 운영한다면 하나하나씩 바꿔줘야할겁니다
- 이를 해결하기 위해 git submodule를 사용해보았습니다.
- git submodule를 활용하면 private한 Githuib 저장소에 민감한 정보를 넣어두고, submodule의 레포 권한이 있는 사람만 민감 정보들을 가져올 수 있게 됩니다.

### 적용법

![스크린샷_2022-07-17_오후_1.51.12](https://tva1.sinaimg.cn/large/e6c9d24egy1h5b77uhfinj21x80hyq59.jpg)

먼저 private 레포를 만들어야 하는 데 저희 dya-mond를 활용하여 만들어서 시큐리티 파일을 업로드 하였습니다. 

이제 amabnb 프로젝트로 돌아와서 submodule를 등록해줍니다. 

```bash
git submodule add https://github.com/dya-mond/submoudle-test.git
```

![스크린샷 2022-07-17 오후 1.53.42.png](https://tva1.sinaimg.cn/large/e6c9d24egy1h5b77xnt8yj20g002w0so.jpg)

![스크린샷 2022-07-17 오후 1.53.00.png](https://tva1.sinaimg.cn/large/e6c9d24egy1h5b780gwyzj20tk060mxm.jpg)

그럼 다음과 같이 .gitmodule파일 생성과 함께 submodule이 등록되어 클론되게 됩니다. 

이후 실행이나 빌드시에 해당 파일을 `resources` 하위에 두기 위해 build.gradle를 수정해줍니다. 

```groovy
processResources.dependsOn('copySubmodule')

task copySubmodule(type: Copy) {
    from './submoudle-test'
    include '*.yaml'
    into './src/main/resources'
}
```

실행, 빌드시에 해당 폴더의 yaml파일이 resources 하위로 들어가게 됩니다. 

여기서 주의해야할 점이 .gitignore로 해당 파일이 업로드 되지 않도록 지정합니다. 

```groovy
// .gitignore
/src/main/resources/application-security.yaml
```

### 팀원들이 해야하는 작업

submodule를 사용하게 되면 팀원들이 해야 하는 작업이 있습니다. 

```bash
git submodule init

git submodule update

git submodule foreach git checkout main
```

- 서브 모듈을 초기화 이후 업데이트한 뒤 main으로 checkout 합니다.
- 이후 변경 사항이 있다면 `git submodule update --remote --merge` 를 쳐주시면 됩니다.

### CI/CD

깃헙 액션을 사용한다면 다음과 같이 submoudle를 가져올 수 있습니다. 

```yaml
- name: Checkout 
		uses: actions/checkout@v1 
		with:
		  token: $ 
		  submodules: true 
```

### 주의점

- submoudle의 변경사항이 있다면 서브모듈 저장소를 먼저 커밋,푸쉬한 뒤 메인 프로젝트에서 커밋,푸쉬 해야합니다.
- 이를 해결하기 위해 서브모듈을 먼저 push하고 메인모듈을 push할 수 있도록 합니다 .

```bash
git push --recurse-submodules=check
git push --recurse-submodules=on-demand
```

```bash
git submodule update
```

- 이후 팀원들은 다음과 같이 update 합니다.

### Reference

[Spring Boot에서 git submodule로 민감 정보(yml) 관리](https://choieungi.github.io/posts/git-submodule/)

[Git - 서브모듈](https://git-scm.com/book/ko/v2/Git-%EB%8F%84%EA%B5%AC-%EC%84%9C%EB%B8%8C%EB%AA%A8%EB%93%88)

[Git 의 서브모듈(Submodule)](https://sgc109.github.io/2020/07/16/git-submodule/)

