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

                echo '2. Archivage des résultats des tests unitaires...'
                junit '**/build/test-results/test/*.xml'

                echo '3. Génération des rapports Cucumber...'
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

                echo '1. Génération du fichier JAR...'
                bat 'gradlew.bat jar'

                echo '2. Génération de la documentation...'
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

                echo 'Déploiement sur myMavenRepo réussi'
            }
        }
    }

    // ========================================
    // Phase 6 : NOTIFICATION
    // ========================================
    post {

        success {
            script {

                // EMAIL
                emailext (
                    subject: "✅ SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                    body: """
                        <h2>Déploiement reussi</h2>
                        <p><b>Projet:</b> ${env.JOB_NAME}</p>
                        <p><b>Build:</b> #${env.BUILD_NUMBER}</p>
                        <p><b>Status:</b> <span style='color:green;'>SUCCESS</span></p>
                        <p><a href='${env.BUILD_URL}'>Voir le build</a></p>
                    """,
                    to: "${env.GMAIL_RECIPIENT}",
                    mimeType: 'text/html'
                )

                // SLACK
                withCredentials([string(
                    credentialsId: 'slack-webhook',
                    variable: 'SLACK_WEBHOOK'
                )]) {
                    bat """
                        curl -X POST -H "Content-type: application/json" ^
                        --data "{\\"text\\":\\"Dploiement réussi : ${env.JOB_NAME} #${env.BUILD_NUMBER}\\"}" ^
                        %SLACK_WEBHOOK%
                    """
                }
            }
        }

        failure {
            script {

                // EMAIL
                emailext (
                    subject: "❌ FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                    body: """
                        <h2>Échec du pipeline</h2>
                        <p><b>Projet:</b> ${env.JOB_NAME}</p>
                        <p><b>Build:</b> #${env.BUILD_NUMBER}</p>
                        <p><b>Status:</b> <span style='color:red;'>FAILED</span></p>
                        <p><a href='${env.BUILD_URL}'>Voir les logs</a></p>
                    """,
                    to: "${env.GMAIL_RECIPIENT}",
                    mimeType: 'text/html'
                )

                // SLACK
                withCredentials([string(
                    credentialsId: 'slack-webhook',
                    variable: 'SLACK_WEBHOOK'
                )]) {
                    bat """
                        curl -X POST -H "Content-type: application/json" ^
                        --data "{\\"text\\":\\"❌ Échec du pipeline : ${env.JOB_NAME} #${env.BUILD_NUMBER}\\"}" ^
                        %SLACK_WEBHOOK%
                    """
                }
            }
        }
    }
}
