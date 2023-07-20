# 11조 프로젝트

<table>
  <tbody>
    <tr>
      <td align="center"><a href="https://hongs429-blog.tistory.com"><img src="https://github.com/hongsh429.png" width=100px;" alt=""/><br /><sub><b>홍승현 </b></sub></a><br /></td>
      <td align="center"><a href="https://soony91.tistory.com"><img src="https://github.com/soon91.png" width="100px;" alt=""/><br /><sub><b>최순</b></sub></a><br /></td>
      <td align="center"><a href="https://deveundol.com/"><img src="https://github.com/nonjk2.png" width="100px;" alt=""/><br /><sub><b>최은석 </b></sub></a><br /></td>
      
  </tbody>
</table>
## 구현

- 회원 기능 (로그인, 회원가입)
- 일상 공유 게시판 기능 (생성, 조회, 수정, 삭제)

## 서비스 : SNS 서비스

> 설명 : 이미지 업로드를 통해 일상을 공유하는 SNS 서비스

## 사용기술

### BACK

<img src="https://img.shields.io/badge/JAVA-007396?style=for-the-badge&logo=java&logoColor=white"><img src="https://img.shields.io/badge/Spring Boot-6DB33F?style=for-the-badge&logo=Spring Boot&logoColor=yellow"><img src="https://img.shields.io/badge/mysql-4479A1?style=for-the-badge&logo=mysql&logoColor=white"><img src="https://img.shields.io/badge/aws-232F3E?style=for-the-badge&logo=aws&logoColor=white">

- Spring Data JPA
- JWT 토큰 + COOKIE 인증으로 사용자 상태 유지
- CICD (GitHub Actions + AWS CodeDeploy + EC2)
- https (CertBot ssl인증서 발급)
- RDS(MySQL)
- AWS S3

### FRONT

<img src="https://img.shields.io/badge/javascript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black"><img src="https://img.shields.io/badge/React-61DAFB?style=for-the-badge&logo=React&logoColor=black">

### ERD

![image](https://github.com/elevenMini/project-back/assets/131283545/b53386ef-5510-4e1e-8e26-ffa42022e18a)

## 시연링크

[링크](https://project-front-rouge.vercel.app/)

## 발생한 이슈와 해결 방법 (트러블 슈팅)

### Back

#### AWS 서비스인 ACM 을 통해 SSL/TLS 인증서 발급 지연

- **문제 상황**: Https 프로토콜을 적용하기 위해선 SSL인증서가 필요했는데, ACM을 통해 인증서를 발급하려고 했으나, 계속되는 지연으로 ACM 서비스를 사용할 수 없었습니다.
- **해결방법**: ACM 외에 무료 인증기관인 Let's Encrypt Certbot을 이용하는 방식으로 바꿨습니다.

#### 도메인 입력할 때, EC2에서 제공한 DNS주소를 입력한 오류

- **문제 상황**: EC2에서 제공하는 DNS 주소는 서버가 내려지면 변경가능한 주소이므로 Certbot에 해당 도메인 주소를 사용할 수 없었습니다. 그리고 여러번의 입력실패가 발생하면, 일정시간 이후에 다시 도메인 주소를 입력해야하는 상황이 자주 발생했습니다.
- **해결방법**: AWS 도메인 구입 후 ROUTE 53 서비스를 이용하여 해당 도메인을 저희 EC2 서버에 연결하여 고정된 도메인 주소를 사용하여 이를 해결 했습니다.<br>
  ![image](https://github.com/elevenMini/HangHaeEleven/assets/131283545/4be7aebd-81ef-4779-9ac8-d1af362a1f87)
<br><br><br><br>

### Front

#### JWT (JSON Web Tokens) 토큰 처리 문제

![네트워크탭](https://user-images.githubusercontent.com/85878391/254797800-e87c1d2e-ae81-402d-9234-c3794ef94780.png)
로그인 이후 서버로부터 토큰을 정상적으로 받아오는 것까지 확인했지만, 이 토큰을 프론트에서 가져와서 HTTP 헤더에 붙여넣는 것에 어려움이 있었습니다.

- 문제의 원인은 브라우저의 보안 업데이트로 인한 쿠키의 `SameSite` 속성의 변경 때문이었습니다. 이 때문에 저희 팀은 각자 조사한 결과 다음의 방법들을 고려하게 되었습니다

  - **방안 1.** `SameSite` 속성을 `None`으로 설정하고, `Secure` 속성을 `true`로 설정하기 (HTTPS 필요)
  - **방안 2.** 모든 요청을 GET 요청으로 변경하기 (`Lax` 속성)
  - **방안 3.** 동일한 도메인에서 배포하기 (route53)
  - **방안 4.** 응답 바디에 토큰을 넣어 프론트에서 직접 토큰을 핸들링하기
  - **방안 5.** nginx를 이용하여 리버스 프록시 설정하기
  - **방안 6.** Docker 활용하기

### 트러블 슈팅 - 해결방안

토큰 문제를 해결하기 위해 CORS (Cross-Origin Resource Sharing) 설정과 `SameSite` 속성을 조정하였습니다.

1. **CORS 설정**

   서버에서는 허용되는 출처를 명시적으로 설정하고, 쿠키를 포함한 요청을 허용하기 위해 `allowCredentials`를 `true`로 설정하였습니다.

   ```tsx
   export const server: AxiosInstance = axios.create({
     baseURL: import.meta.env.VITE_SERVER_ENDPOINT,
     withCredentials: true, // 요청에 쿠키담기
   });
   ```

   - 스프링부트에서 CORS 설정
   - 백엔드에서는 CORS 설정을 변경하여 프론트엔드에서 보낸 쿠키를 수락할 수 있게 해야 함

   ```java
   @Override
   public void addCorsMappings(CorsRegistry registry) {
      registry.addMapping("/api/**")
              .allowedOrigins(ALLOWED_ORIGINS.toArray(new String[0]))
              // 프론트도메인 // * 는 브라우저에서 위험으로 판단하여 다막아버림
              .allowedMethods("*")
              .allowCredentials(true)
              .maxAge(3000);
   }
   ```

2. **`SameSite` 속성 조정**
   ![네트워크탭2](https://user-images.githubusercontent.com/85878391/254797810-58b73b91-fa49-482d-a81b-8b2e60510730.png)
   크롬 버전 80 이후로 쿠키의 `SameSite` 속성은 기본적으로 `Lax`로 설정되어 있어, 이를 `None`으로 설정하였습니다. `None` 설정을 위해서는 `Secure` 속성이 `true`로 설정되어야 하며, HTTPS을 적용해야했습니다.

   ```jsx
   import javax.servlet.http.Cookie;
   import javax.servlet.http.HttpServletResponse;

   public void createCookie(HttpServletResponse response) {
       Cookie cookie = new Cookie("cookieName", "cookieValue");
       cookie.setPath("/");
       cookie.setHttpOnly(true); // 프론트에서 확인 불가. XSS 공격에 방어력 +1
       cookie.setSecure(true);  // SameSite=None 설정을 사용하려면 Secure도 true로 설정해야 함.
       cookie.setAttribute("SameSite", "None");

   		res.addCookie(cookie); // SameSite=None 추가
   }
   ```

   이처럼 브라우저에서 나는 에러들도 백엔드분들과 협력하여 더욱 다양한 시선에서 바라볼수 있어 좋았고, 해결역시 빠르게 해결되어 잘 마무리되었습니다.

## 아쉬운점 & 개선점

### 아쉬운 점

- CICD 구축하는 과정에서의 아쉬움

CICD를 구현하기 위해 Github Actions, AWS CodeDeploy, S3를 사용하였습니다. 크게 어려움이 없었으나, AWS에서 IAM 역할을 설정해주는 단계에서 새로 만든 IAM 계정이 연결이 안되는 문제가 있었습니다. 이를 위해 root계정을 통해 진행하여 CICD를 구축하는데 성공은 했지만, 이후에는 항해 강의와 레퍼런스를 통해 AWS의 IAM 계정 사용을 조금 더 공부해야하겠다고 생각했습니다.
<br><br>

## 새로운 시도

S3와 RDS를 활용하여 이미지를 업로드하고 서비스 이용자에게 이미지를 보여주는 기능을 구현하였습니다. 이 과정에서 form-data와 multipart form-data의 전송방식의 차이를 알게 되었습니다<br>
.![multipartformdata](https://github.com/hongsh429/project-back-origin/assets/131283545/607761e8-25bb-4d83-a3fd-0ae1949598b7)
<br>

S3에 이미지 파일을 사용자에게 보여주기 위해서 file 데이터 저장 당시에 파일의 메타정보인 contentDisposition 속성값을 inline으로 설정해 주어야 file을 다운로드 받지 않고, 이미지를 사용자에게 보여줄 수 있다는 것을 알게 되었습니다.<br>
![contentDisposition](https://github.com/hongsh429/project-back-origin/assets/131283545/3522af00-3d99-4950-9bdf-e72610a21e46)
<br>
<br><br>

## 시간이 있었다면 도전해 볼 내용

### Back

- Refresh 토큰과 access 토큰을 적용하여 사용자의 정보를 계속 유지하는 방법을 적용해보고 싶습니다. 현재는 단순히 access 토큰을 사용하여 사용자의 상태를 서버가 알게 되지만, access 토큰이 만료된 이후는 다시 로그인을 해야합니다. 이를 해결하기 위해 access토큰에 비해 상대적으로 만료기간이 긴 refresh 토큰을 적용해보려고 했었으나, 시도해보지 못했습니다.

<br><br><br>

### Front

1. 이미지를많이 업로드하는만큼 이미지의 크기를 줄이고 업로드를 하는 로직을 추가 **서버 부담 감소 및 브라우저 로딩감소**
2. 이미지를 많이 불러오는 만큼 이미지가 각각 로드되는 속도 역시 데이터가 많아질수록 느려짐, `lazyloading`이나 `prefetch`를 통한 이미지 최적화 로직 등 추가하여 **사용자 경험 상승**
3. 백엔드 서버는 S3와 EC2로 배포되는반면 프론트는 vercel로 배포함 , 나중에 route53을 통한 동일한 도메인으로 배포를 하여 위 트러블을 피해보자
4. 여러 list들이 많은만큼 페이지네이션이나 인피니티스크롤의 사용이 필수가 됨

## 이번주차 느낀점

- 회의를 더욱 자주할것 -> 주기 적인 회의를 통해 서로의 진행상황과 목표치를 생각할 수 있음
- 조금 더 새로운 기술을 시험해보고 싶었지만, 시간적으로나 아직 실력적으로 부족했던 부분이 있었다. 다음에는 조금 더 과감하게 새로운 기술 도입을 해봐야 겠다.
- 막히는 것을 계속 붙잡고 있는 것 보다 잠시 다른 작업을 하거나 다음날 다시 했을 때 새로운 시선으로 문제를 해결할 수 있었어서 마냥 시간을 쓰는게 능사는 아니라는 것을 느꼈음<br>
**이상입니다 감사합니다**
