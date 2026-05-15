---
inclusion: auto
---

# 배포 및 서버 관리 가이드

## git remote 설정
이 워크스페이스의 origin remote는 `https://github.com/developTripbtoz/jooyeonh.git`입니다.
App Runner 서비스가 이 레포에 연결되어 있으므로, 반드시 이 레포로 push해야 배포가 트리거됩니다.

## 서버 제어 명령어
```bash
bash scripts/server-control.sh start   # 서비스 시작 (resume)
bash scripts/server-control.sh stop    # 서비스 중지 (pause)
bash scripts/server-control.sh deploy  # 수동 배포 트리거
```

## 수동 배포 (CLI 직접 실행 시)
`.env` 환경변수를 CLI에 전달하려면 반드시 `set -a` / `set +a`로 감싸야 합니다.
`source .env`만으로는 변수가 export되지 않아 AWS CLI가 인증 정보를 인식하지 못합니다.

```bash
set -a && source .env && set +a
aws apprunner start-deployment --service-arn "$SERVICE_ARN" --region "$AWS_DEFAULT_REGION"
```

## 필요한 IAM 권한
```json
{
  "Action": [
    "apprunner:PauseService",
    "apprunner:ResumeService",
    "apprunner:DescribeService",
    "apprunner:ListServices",
    "apprunner:StartDeployment"
  ]
}
```

## App Runner 런타임 주의사항
- 서비스 생성 시 설정된 런타임(python311 등)은 이후 apprunner.yaml을 변경해도 바뀌지 않음
- 런타임을 변경하려면 서비스를 새로 만들거나, 현재 런타임에 맞춰 코드를 작성해야 함
- 이 서비스는 python311 런타임으로 생성되어 있음

## 일반 배포 흐름
1. 코드 수정 후 `git push` → App Runner 자동 배포
2. 자동 배포가 안 될 경우 → `bash scripts/server-control.sh deploy`로 수동 트리거
