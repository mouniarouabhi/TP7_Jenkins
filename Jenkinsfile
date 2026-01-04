pipeline {
    agent any

    stages {
        stage('Test') {
            steps {
                echo 'Running tests...'
                // Tests unitaires
                sh './gradlew test'

                // Archivage des résultats
                junit '**/build/test-results/test/*.xml'

                // Rapports Cucumber
                cucumber buildStatus: 'SUCCESS',
                         reportTitle: 'Cucumber Report',
                         fileIncludePattern: '**/*.json',
                         jsonReportDirectory: 'reports'
            }
        }

        stage('Code Analysis') {
            steps {
                echo 'Analyzing code quality...'
                // Analyse SonarQube
                sh './gradlew sonarqube'
            }
        }

        stage('Code Quality') {
            steps {
                echo 'Checking Quality Gates...'
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build') {
            steps {
                echo 'Building JAR...'
                // Génération du JAR
                sh './gradlew jar'

                // Génération de la documentation
                sh './gradlew javadoc'

                // Archivage
                archiveArtifacts artifacts: '**/build/libs/*.jar', fingerprint: true
                archiveArtifacts artifacts: '**/build/docs/**', fingerprint: true
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying to MyMavenRepo...'
                sh './gradlew publish'
            }
        }
    }

    post {
        success {
            echo 'Pipeline succeeded!'
            // Notifications email et Slack
            mail to: 'votre-email@example.com',
                 subject: "SUCCESS: ${env.JOB_NAME} - ${env.BUILD_NUMBER}",
                 body: "Le déploiement a réussi. Voir: ${env.BUILD_URL}"
        }
        failure {
            echo 'Pipeline failed!'
            mail to: 'votre-email@example.com',
                 subject: "FAILED: ${env.JOB_NAME} - ${env.BUILD_NUMBER}",
                 body: "Le pipeline a échoué. Voir: ${env.BUILD_URL}"
        }
    }
}