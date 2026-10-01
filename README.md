# CI/CD: GitHub -> Jenkins -> Vercel (+ Telegram)

## Credentials cần tạo trong Jenkins (kiểu Secret text)
| ID | Giá trị |
|---|---|
| vercel-token | Vercel token (vercel.com -> Settings -> Tokens) |
| vercel-org-id | orgId trong .vercel/project.json (chạy `vercel link`) |
| vercel-project-id | projectId trong .vercel/project.json |
| telegram-bot-token | Token từ @BotFather |
| telegram-chat-id | Chat ID từ https://api.telegram.org/bot<TOKEN>/getUpdates |

## Jenkins
1. New Item -> Pipeline
2. Tick "GitHub hook trigger for GITScm polling"
3. Pipeline script from SCM -> Git -> URL repo -> Branch */main -> Script Path: Jenkinsfile

## GitHub Webhook
Repo -> Settings -> Webhooks -> Add webhook
- Payload URL: http://<jenkins-url>/github-webhook/
- Content type: application/json
- Event: Just the push event

## Máy chạy Jenkins cần có
Node.js (npm/npx), git, curl
