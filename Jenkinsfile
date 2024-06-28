pipeline {
    agent any

    environment {
        // Define environment variables and secrets here
        AWS_ACCESS_KEY_ID = credentials('aws-access-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
        S3_BUCKET = 'dev-23'
        EB_APP_NAME = 'angular-todo'
        EB_ENV_NAME = 'angular-todo-env'
        REGION = 'us-west-1'
    }

    tools {
        nodejs 'NodeJS 22.3.0'
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/anand1590/angular-todo.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build') {
            steps {
                sh 'npm run build --prod'
            }
        }

        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: 'dist/**', fingerprint: true
            }
        }

        stage('Upload to S3') {
            steps {
                script {
                    s3Upload(bucket: "${env.S3_BUCKET}", path: 'angular-todo/', workingDir: 'dist/angular-todo', includePathPattern: '**/*')
                }
            }
        }

        stage('Deploy to Elastic Beanstalk') {
            steps {
                script {
                    def versionLabel = "v${env.BUILD_NUMBER}"
                    sh """
                        aws elasticbeanstalk create-application-version --application-name ${env.EB_APP_NAME} --version-label ${versionLabel} --source-bundle S3Bucket=${env.S3_BUCKET},S3Key=angular-todo/
                        aws elasticbeanstalk update-environment --environment-name ${env.EB_ENV_NAME} --version-label ${versionLabel}
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
