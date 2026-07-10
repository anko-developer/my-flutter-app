pipeline {
  agent any

  environment {
    FLUTTER_HOME        = '/opt/flutter'
    PATH                = "${FLUTTER_HOME}/bin:${PATH}"
    AWS_DEFAULT_REGION  = 'ap-northeast-2'
    // ⚠️ 4-cloudfront-setup.sh 실행 후 출력된 ID로 교체
    S3_STAGING   = 's3://my-app-staging-20260709'
    S3_PROD      = 's3://my-app-prod-20260709'
    CF_STAGING   = 'E2JZAKO849DFPD'
    CF_PROD      = 'E1MQFNQZUH7QU9'
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Test') {
      steps {
        sh 'flutter pub get && xvfb-run flutter test'
      }
    }

    stage('Build') {
      steps {
        sh 'flutter build web --release'
      }
    }

    stage('Deploy to Staging') {
      when { branch 'develop' }
      steps {
        sh "aws s3 sync build/web/ ${S3_STAGING} --delete"
        sh """
          ID=\$(aws cloudfront create-invalidation \
            --distribution-id ${CF_STAGING} \
            --paths '/*' \
            --query 'Invalidation.Id' --output text)
          aws cloudfront wait invalidation-completed \
            --distribution-id ${CF_STAGING} --id \$ID
        """
      }
    }

    stage('Deploy to Production') {
      when { branch 'main' }
      steps {
        sh "aws s3 sync build/web/ ${S3_PROD} --delete"
        sh """
          ID=\$(aws cloudfront create-invalidation \
            --distribution-id ${CF_PROD} \
            --paths '/*' \
            --query 'Invalidation.Id' --output text)
          aws cloudfront wait invalidation-completed \
            --distribution-id ${CF_PROD} --id \$ID
        """
      }
    }
  }

  post {
    success { echo '✅ Pipeline succeeded.' }
    failure { echo '❌ Pipeline FAILED — check logs above.' }
  }
}
