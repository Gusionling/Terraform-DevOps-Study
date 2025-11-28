
**태그:** #Terraform #IaC #Study #Variables

# Terraform Input Variables (입력 변수)

## 1. 개요

- **목적**: 테라폼 코드를 하드코딩하지 않고, **동적(Dynamic)** 이고 **재사용(Reusable)** 가능하게 만들기 위함.
    
- **활용**: 동일한 코드로 변수값만 바꿔서 **Dev, QA, Prod** 등 여러 환경을 구축할 때 필수적임.
    

## 2. 변수 선언 (Declaration)

보통 `variables.tf` 또는 `vars.tf` 파일에 모아서 관리한다.

```Terraform
# variables.tf

variable "instance_type" {
  description = "EC2 인스턴스 타입 정의"
  type        = string
  default     = "t2.micro" # 기본값 설정 (없어도 됨)
}
```

- 변수 명세서이다. 
- **참조 방법**: 코드 내에서 `var.변수명`으로 사용 (예: `var.instance_type`)
    

## 3. 변수 값 할당 및 우선순위

변수 값을 설정하는 방법은 여러 가지가 있으며, 상황에 따라 오버라이딩(덮어쓰기) 된다.
**"빈칸에 값을 채우는 방법이 여러 개인데, 겹치면 누가 이기냐?"** 
### A. 기본값 (Default)

`variable` 블록 내에 `default`가 있으면 그 값을 사용.

### B. `terraform.tfvars` 파일 (자동 로드)

프로젝트 루트에 이 파일이 있으면 테라폼이 **자동으로** 값을 읽어들인다.

```Terraform
# terraform.tfvars
instance_type = "t3.micro"
```

### C. 커스텀 tfvars 파일 (환경별 관리)

환경별로 파일을 따로 만들 수 있다 (예: dev.tfvars, prod.tfvars)
환경별로 인스턴스 타입등 리소스를 다르게 사용할 확률이 높기 때문

단, 자동으로 읽지 않으므로 실행 시 플래그를 줘야 한다.

```Bash
# dev.tfvars 파일을 사용하여 실행
terraform apply -var-file="dev.tfvars"
```

### D. CLI 명령어 (`-var`)

명령어 실행 시 직접 값을 주입. 가장 높은 우선순위를 가짐.


```Bash
terraform apply -var "instance_type=t4g.micro"
```

> [!TIP] 우선순위 요약
> 
> CLI (-var) > 파일 지정 (-var-file) > terraform.tfvars > variables.tf의 default

## 4. 데이터 타입 (Data Types)

### 기본 타입 (Primitive)

- `string`: 문자열 (예: "hello")
    
- `number`: 숫자 (정수, 소수 등)
    
- `bool`: 불리언 (true, false)
    

### 복합 타입 (Complex)

- **`map`**: Key-Value 쌍 구조.
    ```Terraform
    variable "instance_map" {
      type = map
      default = {
        dev  = "t2.micro"
        prod = "t3.large"
      }
    }
    ```
    
    - 접근법 1: `var.instance_map["dev"]`
        
    - 접근법 2: `var.instance_map.dev`
        
- `list` (Array/Tuple): 순서가 있는 배열.
    
- `set`: 고유한 값들의 집합 (중복 허용 X).
    
- `object`: 복잡한 구조의 맵 (이름, 나이 등).
    

### 특수 타입

- `null`: 값이 없음을 의미.
    
- `any`: 타입 지정 없음 (유연하지만 권장하지 않음).
    

## 5. 실습 및 디버깅 팁

- **Terraform Console**: 변수 값이 제대로 들어갔는지 확인할 때 유용하다
    
    ```Bash
    $ terraform console
    > var.instance_type
    "t2.micro"
    
    # 특정 파일 적용하여 확인
    $ terraform console -var-file="dev.tfvars"
    > var.instance_type
    "t3.micro"
    ```
    
- **파일 구조화**: 코드가 길어지면 파일을 분리하여 관리하는 것이 좋다.
    
    - `main.tf` (리소스)
        
    - `variables.tf` (변수 선언)
        
    - `outputs.tf` (출력값)
        
    - `terraform.tfvars` (변수 값)
        

---
# Terraform Output Values (출력값)

**태그:** #Terraform #IaC #Study #Outputs

## 1. 개요

- **정의**: 인프라 배포가 완료된 후, 테라폼이 사용자에게 **반환해주는 값**.
    
- **목적**:
    
    1. **정보 확인**: AWS 콘솔에 들어가지 않고 터미널에서 바로 중요 정보(IP 주소, DNS 이름 등)를 확인하기 위함.
        
    2. **모듈 간 데이터 전달**: (심화) 한 모듈의 결과값을 다른 모듈의 입력값으로 넘겨줄 때 사용.
        

## 2. 파일 분리 및 문법

보통 `outputs.tf` 파일을 따로 만들어 관리한다.

```Terraform
# outputs.tf

output "public_ip" {
  description = "생성된 EC2 인스턴스의 공인 IP 주소"
  value       = aws_instance.example.public_ip
}
```

## 3. 속성(Attributes) vs 인수(Arguments)

공식 문서를 볼 때 이 둘을 구별하는 것이 매우 중요하다.

|**구분**|**영어 표기**|**설명**|**방향**|
|---|---|---|---|
|**인수**|**Arguments**|리소스를 **만들 때** 우리가 설정하는 값 (예: `ami`, `instance_type`)|User -> AWS|
|**속성**|**Attributes**|리소스가 **만들어진 후** AWS가 부여하는 값 (예: `public_ip`, `id`)|AWS -> User|

> [!NOTE] 문서 확인 팁
> 
> aws_instance 문서를 볼 때, Argument Reference 섹션은 입력값이고, 스크롤을 맨 아래로 내리면 있는 Attribute Reference 섹션이 우리가 output으로 뽑아낼 수 있는 값들이다.

## 4. 참조 문법 (Reference Syntax) 총정리

테라폼에서 다른 값을 가져올 때 사용하는 문법 패턴이다.

1. **리소스 속성 참조 (이번 강의 핵심)**
    
    - 문법: `<RESOURCE_TYPE>.<NAME>.<ATTRIBUTE>`
        
    - 예시: `aws_instance.example.public_ip`
        
2. **변수 참조**
    
    - 문법: `var.<NAME>`
        
    - 예시: `var.instance_type`
        
3. **데이터 소스 참조**
    
    - 문법: `data.<RESOURCE_TYPE>.<NAME>.<ATTRIBUTE>`
        
    - 예시: `data.aws_ami.ubuntu.id`
        

## 5. 실행 결과 확인

`terraform apply`를 실행하면, 리소스 생성 완료 후 마지막에 출력값을 보여준다.

```Bash
# 실행 예시
Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

Outputs:

public_ip = "3.123.45.67"
```

- **참고**: `(known after apply)`는 "만들어져 봐야 안다"는 뜻이다. (IP 주소는 생성되기 전엔 모르기 때문)
    

---

# Terraform State (상태 관리)

**태그:** #Terraform #State #IaC #Ops

## 1. State란 무엇인가?

테라폼이 관리하는 **인프라의 현재 상태를 기록한 지도(Map)** 이다. 

- **저장 위치**: 기본적으로 `terraform.tfstate`라는 **로컬 JSON 파일**에 저장된다. 
	- 하지만 terraform.tfstate에 저장하지 않고 AWS S3 와 같은 다른 곳에 저장하는 것이 권장된다. 
    
- **역할**:
    1. **매핑 (Mapping)**: 코드 상의 리소스(`resource "aws_instance" "web"`)와 실제 클라우드 상의 리소스(`i-123456789`)를 연결해준다.
        
    2. **변경 감지**: 코드를 수정하고 `plan`을 돌렸을 때, 이 파일과 비교하여 무엇이 바뀌었는지 판단한다.
        

## 2. 동작 원리 (`terraform plan` 시 일어나는 일)

테라폼은 다음 3가지를 비교하여 할 일을 결정한다. 

1. **내 코드 (`.tf`)**: 내가 원하는 상태 (Desired State)
    
2. **State 파일 (`.tfstate`)**: 테라폼이 알고 있는 마지막 상태
    
3. **실제 인프라 (Real World)**: AWS에 실제로 떠 있는 상태
    

> **예시 상황**: 만약 AWS 콘솔에서 몰래 EC2를 삭제했다면?
> 
> - State 파일에는 "있음"으로 기록됨.
>     
> - 실제 AWS 확인 결과 "없음".
>     
> - 테라폼 결론: "어? 없어졌네? 다시 만들어야겠다(Create)."
>     

## 3. 리소스 이름 변경하기 (`terraform state mv`)

코드에서 리소스의 **이름**을 바꾸면 테라폼은 이를 "삭제 후 재생성"으로 인식한다. 
운영 중인 DB나 서버가 날아가버리게 되는 것이다. 이때 사용하는 것이 `state mv` 명령어이다.

**[상황]** `aws_instance.example`을 `aws_instance.web`으로 이름을 바꾸고 싶다.

**[나쁜 방법: 코드만 수정]**

1. 코드에서 `example` -> `web`으로 수정.
    
2. `terraform apply` 실행.
    
3. **결과**: 기존 서버 **삭제(Destroy)** 후 새 서버 **생성(Create)**. (IP 바뀌고 다운타임 발생)
    

**[좋은 방법: State 이동]**

1. 코드에서 `example` -> `web`으로 수정.
    
2. 터미널에서 명령어 실행:
    
    ```Bash
    terraform state mv aws_instance.example aws_instance.web
    ```
    
3. `terraform plan` 실행.
    
4. **결과**: "No changes" (테라폼이 이름만 바뀐 걸 인지하고 서버를 유지함 )
    

## 4. 주요 State 명령어

- `terraform state list`: 현재 관리 중인 리소스 목록 보기.
    
- `terraform state show <리소스명>`: 특정 리소스의 상세 정보 보기.
    
- `terraform state rm <리소스명>`: 리소스를 테라폼 관리에서 **제외**하기 (실제 리소스는 삭제 안 됨).
    
- `terraform state mv <A> <B>`: 리소스 이름 변경 또는 이동.
    

## 5. 주의사항 및 보안 (Security)

### 1) 절대 손으로 편집하지 말 것

`terraform.tfstate`는 JSON 파일이지만, 사람이 직접 수정하다가 괄호 하나라도 틀리면 상태가 꼬여서 복구가 매우 힘들다. 반드시 CLI 명령어(`terraform state ...`)를 사용해서 고쳐라

### 2) 비밀 정보 노출 위험 ⚠️

`tfstate` 파일에는 모든 정보가 **평문(Plain Text)** 으로 저장됩니다.

- DB 비밀번호, AWS Secret Key 등이 그대로 보이는 것이다.
    
- **절대** 이 파일을 GitHub 같은 공개된 저장소에 올리면 안 된다. (`.gitignore` 필수)
    

### 3) 백업과 롤백

테라폼은 `terraform.tfstate.backup` 파일이나 `serial` 번호를 통해 이전 상태를 저장한다.

- 작업이 잘못되었을 때, 백업 파일의 이름을 `terraform.tfstate`로 돌려놓으면 이전 상태로 롤백할 수 있다.