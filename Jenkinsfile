pipeline {
    agent any

    triggers {
        githubPush()   // Kích hoạt khi GitHub gửi webhook push
    }

    environment {
        VERCEL_TOKEN       = credentials('vercel-token')
        VERCEL_ORG_ID      = credentials('vercel-org-id')
        VERCEL_PROJECT_ID  = credentials('vercel-project-id')
        TELEGRAM_BOT_TOKEN = credentials('telegram-bot-token')
        TELEGRAM_CHAT_ID   = credentials('telegram-chat-id')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                script {
                    env.REPO_NAME    = env.GIT_URL.tokenize('/').last().replace('.git', '')
                    env.BRANCH_NAME_ = (env.GIT_BRANCH ?: 'main').replaceFirst('origin/', '')
                    env.COMMIT_SHORT = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    env.COMMIT_MSG   = sh(script: 'git log -1 --pretty=%s', returnStdout: true).trim()

                    sendTelegram("""🚀 Bắt đầu deploy website
Repository: ${env.REPO_NAME}
Branch: ${env.BRANCH_NAME_}
Commit: ${env.COMMIT_SHORT} - ${env.COMMIT_MSG}""")
                }
            }
        }

        stage('Deploy to Vercel') {
            steps {
                script {
                    try {
                        def out = sh(
                            script: 'npx --yes vercel@latest deploy --prod --yes --token "$VERCEL_TOKEN" 2>deploy_err.log',
                            returnStdout: true
                        ).trim()
                        env.SITE_URL = out.readLines().last().trim()
                    } catch (e) {
                        def err = readFile('deploy_err.log').trim()
                        env.ERROR_MSG = err.length() > 500 ? err[-500..-1] : err
                        throw e
                    }
                }
            }
        }
    }

    post {
        success {
            script {
                sendTelegram("""✅ Deploy thành công
Repository: ${env.REPO_NAME}
Branch: ${env.BRANCH_NAME_}
Website: ${env.SITE_URL}""")
            }
        }
        failure {
            script {
                def err = env.ERROR_MSG ?: "Xem chi tiết: ${env.BUILD_URL}console"
                sendTelegram("""❌ Deploy thất bại
Repository: ${env.REPO_NAME}
Branch: ${env.BRANCH_NAME_}
Commit: ${env.COMMIT_SHORT}
Error: ${err}""")
            }
        }
    }
}

// Gửi tin nhắn qua file để tránh lỗi ký tự đặc biệt / dấu nháy
def sendTelegram(String text) {
    writeFile file: 'tg_msg.txt', text: text
    sh '''
        curl -s -X POST "https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendMessage" \
             -d chat_id="${TELEGRAM_CHAT_ID}" \
             --data-urlencode "text@tg_msg.txt" > /dev/null
    '''
}
