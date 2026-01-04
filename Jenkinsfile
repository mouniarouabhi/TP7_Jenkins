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
                bat './gradlew.bat test'

                echo '2. Archivage des résultats des tests unitaires...'
                junit '**/build/test-results/test/*.xml'

                echo '3. Génération des rapports de tests Cucumber...'
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
                echo 'Analyse de la qualité du code avec SonarQube...'
                bat './gradlew.bat sonar'
            }
        }

        // ========================================
        // Phase 3 : CODE QUALITY
        // ========================================
        stage('Code Quality Gate') {
            steps {
                echo '========== Phase Code Quality =========='
                echo 'Vérification de l\'état de Quality Gates...'
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
                echo 'Quality Gate : PASSED ✓'
            }
        }

        // ========================================
        // Phase 4 : BUILD
        // ========================================
        stage('Build') {
            steps {
                echo '========== Phase Build =========='

                echo '1. Génération du fichier JAR...'
                bat './gradlew.bat jar'

                echo '2. Génération de la documentation...'
                bat './gradlew.bat javadoc'

                echo '3. Archivage du fichier JAR et de la documentation...'
                archiveArtifacts artifacts: '**/build/libs/*.jar',
                                 fingerprint: true,
                                 allowEmptyArchive: false

                archiveArtifacts artifacts: '**/build/docs/javadoc/**',
                                 fingerprint: true,
                                 allowEmptyArchive: false
            }
        }

        // ========================================
        // Phase 5 : DEPLOY
        // ========================================
        stage('Deploy') {
            steps {
                echo '========== Phase Deploy =========='
                echo 'Déploiement du fichier JAR sur MyMavenRepo...'
                bat './gradlew.bat publish'
                echo 'Déploiement réussi ✓'
            }
        }
    }

    // ========================================
    // Phase 6 : NOTIFICATION
    // ========================================
    post {
        success {
            echo '========== Phase Notification (SUCCESS) =========='
            script {
                // Notification par Email
                emailext (
                    subject: "✅ SUCCESS: Pipeline ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                    body: """
                        <h2>Déploiement réussi !</h2>
                        <p><strong>Projet:</strong> ${env.JOB_NAME}</p>
                        <p><strong>Build:</strong> #${env.BUILD_NUMBER}</p>
                        <p><strong>Status:</strong> <span style="color:green;">SUCCESS</span></p>
                        <p><strong>URL:</strong> <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                        <hr>
                        <p>Toutes les phases du pipeline ont été exécutées avec succès.</p>
                    """,
                    to: 'mm_rouabhi@esi.dz',
                    from: 'jenkins@esi.dz',
                    mimeType: 'text/html'
                )

                // Notification Slack
                bat """
                    curl -X POST -H "Content-type: application/json" --data "{\\"text\\":\\"🚀 Déploiement réussi: ${env.JOB_NAME} #${env.BUILD_NUMBER}\\"}" https://hooks.slack.com/services/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
                """
            }
            echo 'Notifications envoyées (Email + Slack) ✓'
        }

        failure {
            echo '========== Phase Notification (FAILURE) =========='
            script {
                // Notification par Email en cas d'échec
                emailext (
                    subject: "❌ FAILED: Pipeline ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                    body: """
                        <h2>Échec du pipeline !</h2>
                        <p><strong>Projet:</strong> ${env.JOB_NAME}</p>
                        <p><strong>Build:</strong> #${env.BUILD_NUMBER}</p>
                        <p><strong>Status:</strong> <span style="color:red;">FAILED</span></p>
                        <p><strong>URL:</strong> <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                        <hr>
                        <p>Une ou plusieurs phases du pipeline ont échoué. Veuillez vérifier les logs.</p>
                    """,
                    to: 'mm_rouabhi@esi.dz',
                    from: 'jenkins@esi.dz',
                    mimeType: 'text/html'
                )

                // Notification Slack en cas d'échec
                bat """
                    curl -X POST -H "Content-type: application/json" --data "{\\"text\\":\\"❌ Échec du pipeline: ${env.JOB_NAME} #${env.BUILD_NUMBER}\\"}" https://hooks.slack.com/services/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
                """
            }
            echo 'Notifications d\'échec envoyées (Email + Slack) ✓'
        }
    }
}