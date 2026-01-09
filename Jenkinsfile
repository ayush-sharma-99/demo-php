pipeline {
    agent { label 'php-agent' }

    environment {
        MAIN_BRANCH = "main"
        DEV_BRANCH  = "developer"
        APP_DIR     = "/opt/app"
        BACKUP_DIR  = "/opt/develop/php1"
    }

    stages {

        stage('Checkout Main Branch') {
            steps {
                git branch: "${MAIN_BRANCH}",
                    url: 'https://github.com/ayush-sharma-99/demo-php.git'
            }
        }

        stage('Fetch Developer Branch') {
            steps {
                sh "git fetch origin ${DEV_BRANCH}"
            }
        }

        stage('Merge Developer into Main') {
            steps {
                sh """
                git config user.email "jenkins@company.com"
                git config user.name  "jenkins"
                git merge origin/${DEV_BRANCH}
                """
            }
        }

        stage('Identify Changed PHP File') {
            steps {
                script {
                    CHANGED_FILE = sh(
                        script: "git diff --name-only HEAD~1 HEAD | grep '.php'",
                        returnStdout: true
                    ).trim()

                    if (!CHANGED_FILE) {
                        error "No PHP file change detected!"
                    }

                    echo "Changed file detected: ${CHANGED_FILE}"
                }
            }
        }

        stage('Backup Existing File') {
            steps {
                sh """
                TIMESTAMP=\$(date +%F-%H-%M-%S)
                LIVE_FILE=${APP_DIR}/${CHANGED_FILE}

                if [ -f \$LIVE_FILE ]; then
                    cp \$LIVE_FILE ${BACKUP_DIR}/\$(basename ${CHANGED_FILE})-\$TIMESTAMP
                else
                    echo "No existing file found, skipping backup"
                fi
                """
            }
        }

        stage('Deploy New File') {
            steps {
                sh """
                mkdir -p ${APP_DIR}/\$(dirname ${CHANGED_FILE})
                cp ${CHANGED_FILE} ${APP_DIR}/${CHANGED_FILE}
                """
            }
        }

        stage('Push Changes to Main') {
            steps {
                sh "git push origin ${MAIN_BRANCH}"
            }
        }
    }

    post {
        success {
            echo "✅ Merge, backup, and deployment completed successfully"
        }
        failure {
            echo "❌ Pipeline failed. No files deployed."
        }
    }
}

