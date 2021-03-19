pipeline {
    agent { node { label 'maven' } }
    stages {
        stage('Test') {
            steps {
                sh './mvnw clean test'
            }
        }
        stage('Build') {
            environment {
                QUAY = credentials('MONITORING_LAB_QUAY_CREDENTIALS')
            }
            steps {
                sh './scripts/include-container-extensions.sh'

                sh '''
                    ./scripts/build-and-push-image.sh \
                    -b $BUILD_NUMBER \
                    -u "$QUAY_USR" \
                    -p "$QUAY_PSW" \
                    -r do400-monitoring-lab
                '''
            }
        }
        stage('Security Scan') {
            steps {
                sh '''
                    oc process -f kubefiles/security-scan-template.yml \
                    -n RHT_OCP4_DEV_USER-monitoring-lab \
                    -p QUAY_USER=gunnar-do \
                    -p QUAY_REPOSITORY=do400-monitoring-lab \
                    -p APP_NAME=calculator \
                    | oc replace --force \
                    -n RHT_OCP4_DEV_USER-monitoring-lab -f -
                '''
                sh '''
                    ./scripts/check-job-state.sh "calculator-trivy" \
                    "ghilling-monitoring-lab"
                '''
            }
        }
        stage('Deploy') {
            steps {
                sh '''
                    ./scripts/redeploy.sh \
                        -d calculator \
                        -n ghilling-monitoring-lab
                '''
            }
        }
    }
    post {
        failure {
            mail to: "team@example.com",
                subject: "Pipeline failed: ${currentBuild.fullDisplayName}",
                body: "The following pipeline failed: ${env.BUILD_URL}"
        }
    }

}
