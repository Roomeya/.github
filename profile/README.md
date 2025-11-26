# Roomeya

> 학생 매칭 및 폼 관리 시스템

Roomeya는 학생들의 정보를 수집하고 매칭하는 서버리스 기반의 폼 관리 시스템입니다.

## 🎯 프로젝트 개요

Roomeya는 AWS 서버리스 아키텍처를 기반으로 구축된 학생 매칭 플랫폼입니다. 사용자는 폼을 생성하고, 학생들은 폼을 제출하며, 시스템은 자동으로 매칭을 처리하고 결과를 이메일로 발송합니다.

### 주요 기능

- 📝 **폼 생성 및 관리**: 커스텀 폼 생성 및 응답 수집
- 👥 **학생 매칭**: 자동화된 매칭 알고리즘
- 📊 **데이터 처리**: 엑셀 파일 업로드 및 처리
- 📧 **이메일 알림**: SES를 통한 자동 이메일 발송
- 🔐 **인증 시스템**: Cognito 기반 OAuth 2.0

## 🏗️ 아키텍처

```
┌─────────────┐
│   사용자    │
└──────┬──────┘
       │
       ▼
┌─────────────────────────────────────────┐
│          Cognito User Pool              │
│         (OAuth 2.0 인증)                │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│        API Gateway (HTTP API)           │
│         (CORS, Routes)                  │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│         Lambda Functions                │
│  ┌────────────────────────────────┐    │
│  │ • CreateForm                   │    │
│  │ • getFormList                  │    │
│  │ • SubmitForm                   │    │
│  │ • upload-url                   │    │
│  │ • identify_student             │    │
│  └────────────────────────────────┘    │
└──────┬──────────────────┬───────────────┘
       │                  │
       ▼                  ▼
┌─────────────┐    ┌─────────────┐
│  DynamoDB   │    │     S3      │
│             │    │             │
│ • Forms     │    │ • Export    │
│ • Responses │    │ • Upload    │
│ • Results   │    │             │
│ • Students  │    └─────────────┘
└─────────────┘
       │
       ▼
┌─────────────────────────────────────────┐
│         Step Functions                  │
│  ┌────────────────────────────────┐    │
│  │ matchingProcessor              │    │
│  │        ↓                       │    │
│  │ excelProcessor                 │    │
│  │        ↓                       │    │
│  │ emailSender                    │    │
│  └────────────────────────────────┘    │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│              SES                        │
│        (이메일 발송)                    │
└─────────────────────────────────────────┘
```

## 📦 레포지토리 구조

### 1. [Roomeya-Infrastructure-Terraform](https://github.com/Roomeya/Roomeya-Infrastructure-Terraform)
AWS 인프라를 Terraform으로 관리하는 레포지토리

**포함 리소스:**
- Lambda Functions (9개)
- DynamoDB Tables (4개)
- S3 Buckets (2개)
- API Gateway (HTTP API)
- Step Functions
- Cognito User Pool
- SES, VPC, CloudWatch

### 2. [Roomeya-Lambda-Functions](https://github.com/Roomeya/Roomeya-Lambda-Functions)
Lambda 함수의 소스 코드를 관리하는 레포지토리

**Lambda 함수 목록:**
- `CreateForm` - 폼 생성
- `getFormList` - 폼 목록 조회
- `SubmitForm` - 폼 제출
- `upload-url` - S3 업로드 URL 생성
- `identify_student` - 학생 식별
- `matchingProcessor` - 매칭 처리
- `matchingResult` - 매칭 결과 조회
- `excelProcessor` - 엑셀 파일 처리
- `emailSender` - 이메일 발송

### 3. [Roomeya-Lambda-Deployments](https://github.com/Roomeya/Roomeya-Lambda-Deployments)
Lambda 함수의 배포용 zip 파일을 관리하는 레포지토리

## 🔄 개발 워크플로우

### 1. Lambda 함수 개발
```bash
# 1. Lambda Functions 레포 클론
git clone https://github.com/Roomeya/Roomeya-Lambda-Functions.git
cd Roomeya-Lambda-Functions

# 2. 함수 수정
cd getFormList
vim lambda_function.py

# 3. 로컬 테스트
python -m pytest tests/

# 4. 커밋 & 푸시
git add .
git commit -m "Update getFormList function"
git push origin main
```

### 2. Lambda 배포
```bash
# 1. 함수 빌드 (zip 생성)
cd Roomeya-Lambda-Functions
./scripts/build.sh

# 2. Deployments 레포에 복사
cp dist/*.zip ../Roomeya-Lambda-Deployments/

# 3. Deployments 레포 푸시
cd ../Roomeya-Lambda-Deployments
git add *.zip
git commit -m "Update Lambda deployments"
git push origin main
```

### 3. 인프라 배포
```bash
# 1. Infrastructure 레포로 이동
cd Roomeya-Infrastructure-Terraform

# 2. Terraform 계획 확인
terraform plan

# 3. 적용
terraform apply

# 4. 변경사항 커밋
git add .
git commit -m "Update infrastructure"
git push origin main
```

## 🚀 빠른 시작

### 사전 요구사항
- AWS CLI 설치 및 설정
- Terraform 설치 (v1.0+)
- Python 3.14
- Git

### 초기 설정

1. **레포지토리 클론**
```bash
git clone https://github.com/Roomeya/Roomeya-Infrastructure-Terraform.git
git clone https://github.com/Roomeya/Roomeya-Lambda-Functions.git
git clone https://github.com/Roomeya/Roomeya-Lambda-Deployments.git
```

2. **AWS 자격 증명 설정**
```bash
aws configure
```

3. **Terraform 초기화**
```bash
cd Roomeya-Infrastructure-Terraform
terraform init
```

4. **기존 리소스 Import (최초 1회)**
```bash
# Lambda 함수들 import
terraform import aws_lambda_function.get_form_list getFormList
terraform import aws_lambda_function.create_form CreateForm
# ... (나머지 리소스들)
```

5. **인프라 배포**
```bash
terraform apply
```

## 🛠️ 기술 스택

### Backend
- **Compute**: AWS Lambda (Python 3.14)
- **API**: API Gateway (HTTP API)
- **Database**: DynamoDB
- **Storage**: S3
- **Orchestration**: Step Functions
- **Auth**: Cognito
- **Email**: SES
- **Monitoring**: CloudWatch

### Infrastructure
- **IaC**: Terraform
- **Version Control**: Git/GitHub
- **CI/CD**: GitHub Actions (예정)

### Frontend (별도 레포)
- React + Vite
- AWS Amplify

## 📊 데이터 흐름

### 폼 생성 플로우
```
사용자 → Cognito 인증 → API Gateway → CreateForm Lambda → DynamoDB (Forms)
```

### 폼 제출 플로우
```
학생 → API Gateway → SubmitForm Lambda → DynamoDB (FormResponses)
```

### 매칭 처리 플로우
```
관리자 → Step Functions 시작
  ↓
matchingProcessor (DynamoDB 조회)
  ↓
excelProcessor (결과 생성 → S3)
  ↓
emailSender (SES 이메일 발송)
```

## 🔐 보안

- **인증**: Cognito User Pool (OAuth 2.0)
- **권한**: IAM Roles (최소 권한 원칙)
- **네트워크**: VPC, Security Groups
- **데이터**: DynamoDB 암호화, S3 버킷 정책
- **비밀 관리**: AWS Secrets Manager (권장)

## 📝 환경 변수

Lambda 함수에서 사용하는 주요 환경 변수:
- `TABLE_NAME`: DynamoDB 테이블 이름
- `BUCKET_NAME`: S3 버킷 이름
- `REGION`: AWS 리전

## 🤝 기여 가이드

### 브랜치 전략
- `main`: 프로덕션 브랜치
- `develop`: 개발 브랜치
- `feature/*`: 기능 개발 브랜치
- `hotfix/*`: 긴급 수정 브랜치

### 커밋 메시지 규칙
```
feat: 새로운 기능 추가
fix: 버그 수정
docs: 문서 수정
style: 코드 포맷팅
refactor: 코드 리팩토링
test: 테스트 추가
chore: 빌드/설정 변경
```

### Pull Request 프로세스
1. Feature 브랜치 생성
2. 코드 작성 및 테스트
3. PR 생성 (develop 브랜치로)
4. 코드 리뷰
5. Merge

## 👥 팀원

<!-- 팀원 정보는 여기에 추가 -->

## 📄 라이센스

MIT License

## 📞 문의

- Organization: [Roomeya](https://github.com/Roomeya)
- Issues: 각 레포지토리의 Issues 탭 활용

---

**Built with ❤️ by Roomeya Team**
