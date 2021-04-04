# IAM

**AWS Identity and Access Management**

> AWS 리소스에 대한 엑세스를 안전하게 제어할 수 있는 웹 서비스
> 사용자가 리소스를 조작 및 제어하는 것. CRUD

- [IAM 서비스 사용하기](#iam-서비스-사용하기)

- [IAM Policy](#iam-policy)

- [IAM Role](#iam-role)

### Authentication

Authentication = 인증

> Authentication이란 자신이 누구라고 주장할 때 이를 확인하는 절차

> 사용자 로그인 : 사용자 ID와 Password를 이용해서 인증을 하는 절차

### Authorization

Authorization = 권한 부여

> Authorization란 사용자가 원하는 리소스에 접근하는 것을 허용하는 과정

### AWS의 인증과 권한 부여

AWS에서 권한 부여의 주체(권한 부여를 담당하는 것)는 `IAM Policy`이고

인증의 주체(인증을 담당하는 것)는 `User(Group)`이다.

- IAM User (Group)

  - 인증의 주체
  - long term credentials (장기 자격 증명)

- IAM Role

  - 인증의 주체
  - temporary term credentials (임시 자격 증명)
    - 만료 기간이 정해져 있다.
  - 다른 AWS들의 서비스들이 권한을 부여 받고 싶을 때 주로 사용
    - EC2가 S3에 접근하고 싶다.
    - Lambda가 S3에 접근하고 싶다.

- IAM Policy

  - 권한 부여의 주체

### IAM Group

- 공통의 권한을 가지는 사용자의 집합
- 그룹을 생성 후 IAM Policy 연결
- 그룹에 사용자 추가
- 그룹 내 사용자는 그룹과 연결된 Policy의 권한을 부여 받는다.

### IAM User

- 그룹의 IAM Policy에 따하 권한을 부여받는다.
- 사용자에게 직접 Policy를 추가할 수도 있다.
- IAM User의 인증방식과 사용용도
- ID / Password 방식 : 관리 콘솔에서 사용한다.
- long term credentials (장기 자격 증명)

<br>

---

<br>

## IAM 서비스 사용하기

### ROOT 사용자 MFA 적용하기

> 반드시 폰(디바이스)을 변경하게 된거나 초기화를 하게 된다면, 반드시 AWS MFA를 삭제하자.

1. AWS CONSOLE ROOT LOGIN

2. 내 보안 자격 증명 (My Security Credentials) 클릭

3. 멀티 팩터 인증 (MFA) 클릭

4. MFA 활성화 클릭

5. 가상 MFA 디바이스 체크

6. Google OTP 앱 다운

7. MFA 코드 입력

   - 시간이 지나면 코드가 변경되는데 연속해서 나오는 코드 두 개를 입력한다.

8. 다시 로그인해서 테스트

### IAM admin 사용자 추가하기

1. ROOT 계정으로 로그인

2. IAM 서비스 접속

3. 그룹 클릭

4. 새로운 그룹 만들기 클릭

   - 그룹 이름은 아무거나 (ex. admin)
   - 정책 연결에서 `AdministratorAccess` 정책 연결
   - 그룹 생성

5. 사용자 클릭

6. 사용자 추가하기 클릭

   - `AWS Management Console 액세스` 체크
   - `비밀번호 재설정 필요` 체크 취소
   - `그룹에 사용자 추가`에서 아까 만든 admin 그룹 체크

7. 사용자 생성이 완료 되었다면, 해당 사용자 정보 csv 다운로드
   - `Console login link`를 사용하여 로그인한다.

### IAM developer 사용자 추가하기

1. IAM 서비스 접속

2. `그룹` 클릭

3. 새로운 그룹 만들기 클릭

   - 그룹 이름은 아무거나 (ex. developer)
   - 정책 연결에서 `PowerUserAccess` 정책 연결
     - 서비스는 사용할 수 있으나, 계정 관련해서는 사용 불가
   - 그룹 생성

4. `사용자` 클릭

5. 사용자 추가하기 클릭

   - `AWS Management Console 액세스` 체크
   - `비밀번호 재설정 필요` 체크 취소
   - `그룹에 사용자 추가`에서 아까 만든 developer 그룹 체크

6. 사용자 생성이 완료 되었다면, 해당 사용자 정보 csv 다운로드
   - `Console login link`를 사용하여 로그인한다.

### 로그인 URL 변경하기

1. IAM 서비스 대시보드

2. `이 계정의 IAM 사용자를 위한 로그인 URL`의 `사용자 지정` 버튼 클릭

<br>

---

<br>

## IAM Policy

- AWS 서비스의 접근 권한을 세부적으로 관리하기 위해 사용
- JSON 포맷의 문서

### IAM Policy의 종류

- **AWS 관리 정책**

  - AWS가 미리 만들어 놓은 정책, 사용자는 편집 불가능

```json
// ex. AdministratorAccess 정책
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "*",
      "Resource": "*"
    }
  ]
}
```

- **사용자 관리 정책**

  - 사용자가 직접 생성한 정책
  - 기존 정책으로부터 생성 및 수정 또는 직접 생성 가능
  - GUI 편집기 / JSON 편집기 모두 사용 가능

- **Inline 정책**

  - 1회성 정책 (재사용 불가)

<br>

---

<br>

## IAM Role

> 주로 AWS 서비스들이 직접 다른 AWS 서비스를 제어하기 위해 사용한다.

> 사용자나 응용 프로그램에서 일시적으로 AWS 리소스에 접근 권한을 얻을 때도 사용한다.

- 특정 개체에게 리소스의 접근 권한을 부여하기 위해 사용한다.
- 특정 개체 (IAM 사용자, AWS 서비스, 다른 계정, AWS 관리 계정)
- Short Term Credential (임시 자격 증명)
- 안전모 아이콘
- assumeRole API

### IAM Role의 주요 구성요소

- Role ARN : 역할을 호출하기 위해 필요

- IAM Policy : 이 역할이 어떤 권한을 부여할 수 있는가

- **신뢰 관계 : 어떤 개체가 IAM Role을 호출할 수 있는가**

- 그 외 유지 기간, 이름 등도 필요
