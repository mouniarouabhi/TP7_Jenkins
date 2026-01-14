pipeline {
    agent any

    stages {

        // ========================================
        // Phase 1 : TEST
        // ========================================
        stage('Test') {
            steps {
                echo '========== Phase Test =========='

                echo '1. Lancement des tests unitaires...'
                bat 'gradlew.bat test'

                echo '2. Archivage des resultats des tests unitaires...'
                junit '**/build/test-results/test/*.xml'

                echo '3. Generation des rapports Cucumber...'
                cucumber buildStatus: 'SUCCESS',
                         reportTitle: 'Cucumber Test Report',
                         fileIncludePattern: '**/*.json',
                         jsonReportDirectory: 'reports'
            }
        }

        // ========================================
        // Phase 2 : CODE ANALYSIS
        // ========================================
        stage('Code Analysis') {
            steps {
                echo '========== Phase Code Analysis =========='

                withSonarQubeEnv('sonar') {
                    withCredentials([string(
                        credentialsId: 'sonar-token',
                        variable: 'SONAR_TOKEN'
                    )]) {
                        bat 'gradlew.bat sonar -Dsonar.login=%SONAR_TOKEN%'
                    }
                }
            }
        }

        // ========================================
        // Phase 3 : CODE QUALITY
        // ========================================
        stage('Code Quality Gate') {
            steps {
                echo '========== Phase Code Quality =========='
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
                echo 'Quality Gate PASSED ✓'
            }
        }

        // ========================================
        // Phase 4 : BUILD
        // ========================================
        stage('Build') {
            steps {
                echo '========== Phase Build =========='

                echo '1. Generation du fichier JAR...'
                bat 'gradlew.bat jar'

                echo '2. Generation de la documentation...'
                bat 'gradlew.bat javadoc'

                echo '3. Archivage des artefacts...'
                archiveArtifacts artifacts: '**/build/libs/*.jar', fingerprint: true
                archiveArtifacts artifacts: '**/build/docs/javadoc/**', fingerprint: true
            }
        }

        // ========================================
        // Phase 5 : DEPLOY
        // ========================================
        stage('Deploy') {
            steps {
                echo '========== Phase Deploy =========='

                withCredentials([usernamePassword(
                    credentialsId: 'maven-repo-creds',
                    usernameVariable: 'MAVEN_USERNAME',
                    passwordVariable: 'MAVEN_PASSWORD'
                )]) {
                    bat 'gradlew.bat publish'
                }

                echo 'Deploiement sur myMavenRepo reussi'
            }
        }
    }

    // ========================================
    // Phase 6 : NOTIFICATION
    // ========================================
    post {

        success {
            script {
                // EMAIL (non-blocking)
                try {
                    emailext(
                        subject: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                        body: """
                            <h2>Deploiement reussi</h2>
                            <p><b>Projet:</b> ${env.JOB_NAME}</p>
                            <p><b>Build:</b> #${env.BUILD_NUMBER}</p>
                            <p><b>Status:</b> SUCCESS</p>
                            <p><a href='${env.BUILD_URL}'>Voir le build</a></p>
                        """,
                        to: "mr_asbar@esi.dz",
                        mimeType: 'text/html',
                        replyTo: "mm_rouabhi@esi.dz"
                    )
                } catch (err) {
                    echo "Email failed (SMTP/network restriction)"
                    echo err.toString()
                }

                // SLACK (blocking is OK)
                withCredentials([string(
                    credentialsId: 'slack-webhook',
                    variable: 'SLACK_WEBHOOK'
                )]) {
                    bat """
                        curl -X POST -H "Content-type: application/json" ^
                        --data "{\\"text\\":\\"Deploiement reussi : ${env.JOB_NAME} #${env.BUILD_NUMBER}\\"}" ^
                        %SLACK_WEBHOOK%
                    """
                }
            }
        }


        failure {
            script {
                try {
                    emailext(
                        subject: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                        body: "Pipeline failed: ${env.BUILD_URL}",
                        to: "mm_rouabhi@esi.dz"
                    )
                } catch (err) {
                    echo "Email failed (SMTP/network restriction)"
                }

                withCredentials([string(
                    credentialsId: 'slack-webhook',
                    variable: 'SLACK_WEBHOOK'
                )]) {
                    bat """
                        curl -X POST -H "Content-type: application/json" ^
                        --data "{\\"text\\":\\"echec du pipeline : ${env.JOB_NAME} #${env.BUILD_NUMBER}\\"}" ^
                        %SLACK_WEBHOOK%
                    """
                }
            }
        }
    }
}
