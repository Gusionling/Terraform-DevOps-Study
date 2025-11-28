## 1. Provider (프로바이더)

> **한마디로:** "누구랑 작업할 거야?" 

테라폼 자체는 AWS가 뭔지, Azure가 뭔지 모른다. 그래서 **"AWS랑 소통할 줄 아는 요소"** 가 **Provider** 이다.

- **역할:** 테라폼의 명령을 해당 클라우드(AWS, Azure 등)가 알아들을 수 있게 **통역**하고 **인증(로그인)** 하는 역할.

```Terraform
# provider.tf
provider "aws" {
  region = "ap-northeast-2"  # "AWS 서울 지점을 쓰겠다"
}
```

## 2. Resource (리소스)

> **개념:** "무엇을 **만들** 것인가?" 

테라폼을 쓰는 진짜 목적이다. 실제로 클라우드에 **생성하고, 수정하고, 관리할** 대상이다. 코드를 작성하고 `apply`를 하면, 이 리소스 블록에 적힌 대로 실제 인프라가 **생겨나는 것이다.**

- **역할:** 인프라 생성 (Create), 수정 (Update), 삭제 (Delete).

```Terraform
# main.tf
resource "aws_instance" "my_server" {
  ami           = "ami-12345678" # 만들 재료 ID
  instance_type = "t2.micro"     # 사이즈
}
```


## 3. Data Source (데이터 소스)

> **개념:** "뭘 **참고(조회)** 할 것인가?" 

**이미 존재하는 정보**를 읽어오기만 할 때 쓴다. 

- **역할:** 정보 조회 (Read Only). 클라우드에 이미 있는 리소스나, 동적으로 변하는 값(예: 최신 리눅스 이미지 ID)을 가져옴.

```Terraform
# main.tf
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]
  # "우분투 최신 버전 ID가 뭐야? 검색해줘" (만드는 게 아님!)
}
```

> **핵심:** `data`는 **참조만** 한다. 테라폼이 이걸 생성하거나 삭제하지 않습니다.


# Terraform Modules (모듈)

앞서 살펴본 **변수(Input), 리소스(Logic), 출력(Output)** 이 합쳐져서 하나의 **"부품"** 이 된 것을 말한다.

## 1. 개념 

> **"관련된 리소스들을 하나로 묶은 패키지"** 

main.tf 파일 하나에 EC2, 보안그룹, VPC 등을 다 때려 넣는다면 코드가 길어지면 관리가 안 된다.

이걸 기능별로 쪼개서 재사용 가능한 덩어리로 만든 것이 모듈이다.


## 2. 왜 쓰는가? (장점)

1. **재사용성 (Reusability)**: "웹 서버 세트(EC2 + 보안그룹 + IP)"라는 모듈을 한 번 만들어두면, 개발팀/운영팀/테스트팀이 가져다 쓰기만 하면 됨.
    
2. **가독성 (Readability)**: `main.tf`가 수백 줄의 리소스 나열이 아니라, 깔끔한 모듈 호출 몇 줄로 바뀜.
    
3. **표준화 (Standardization)**: 팀 전체가 똑같은 보안 규칙이 적용된 인프라를 생성하도록 강제할 수 있음.
    

## 3. 모듈의 3요소 

모듈은 **입력**을 받아서 **리소스**를 만들고 **결과**를 뱉는다. 

| **테라폼 파일**         | **모듈에서의 역할**    |
| ------------------ | --------------- |
| **`variables.tf`** | **입력 (Input)**  |
| **`main.tf`**      | **본문 (Body)**   |
| **`outputs.tf`**   | **출력 (Output)** |

**루트 모듈 (Root Module)**

**테라폼 코드를 작성하고 있는 그 폴더**도 모듈이다.

- 테라폼을 실행하는 현재 디렉토리를 **루트 모듈(Root Module)** 이라 부른다.
    
- 나중에는 루트 모듈이 자식 모듈(Child Module)을 부르는 구조가 된다.


모듈은 Terraform Registry에서 가져올 수도 있다. 예를 들어 VPC 를 그냥 terraform 에서 정의하려면 명세해야 하는 것들이 많은데, Terraform Registry 에 있는 vpc 모듈을 사용하면 간단하게 정의할 수 있다. 

[VPC module](https://registry.terraform.io/modules/terraform-aws-modules/vpc/aws/latest)

