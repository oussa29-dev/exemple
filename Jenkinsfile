pipeline {
  agent any
  stages {
    stage('Test') {
      steps {
        echo 'Running tests with Gradle'
        // sh 'chmod+x gradlew'
        sh './gradlew test'
        junit '**/build/test-results/test/*.xml'
        cucumber '**/reports/*.json'
      }
    }

    stage('Code Analysis') {
      steps {
        echo 'Analyse code with SonarQube'
        script {
          withSonarQubeEnv('sonar') {
            sh './gradlew sonar --stacktrace'
          }
        }

      }
    }

    stage('Code Quality') {
      steps {
        script {
          def qg = waitForQualityGate()
          if (qg.status != 'OK') {
            error "Pipeline failed due to Quality Gate status: ${qg.status}"
          }
        }

      }
    }

    stage('Build') {
      steps {
        echo 'Running Build...'
        sh './gradlew jar'
        sh './gradlew javadoc'
        archiveArtifacts(artifacts: 'build/reports/tests/test/index.html', allowEmptyArchive: true)
        archiveArtifacts(artifacts: 'build/libs/*.jar', allowEmptyArchive: true)
      }
    }

    stage('Deploy') {
      steps {
        echo 'Start Deployement on MyMavenRepo...'
        sh './gradlew publish'
      }
    }

    stage('Notification') {
      steps {
        echo 'Send Notifications To the Team...'
        mail(subject: 'integration', body: 'Déploiement réussi', to: 'ko_benkhaoua@esi.dz')
        slackSend(message: 'Déploiement réussi', sendAsText: true)
      }
    }

  }
  environment {
    sonarQube = 'sonar'
  }
  post {
    failure {
      echo 'Pipeline Failed'
      mail(subject: 'ERREUR', body: 'ERREUR', to: 'ko_benkhaoua@esi.dz')
      slackSend(message: 'ERREUR', sendAsText: true)
    }

  }
}
