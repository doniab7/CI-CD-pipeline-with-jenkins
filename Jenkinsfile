pipeline {
  agent {
      docker {
        image 'docker:20.10.24-dind'
        args '-v /var/run/docker.sock:/var/run/docker.sock'
      }
    }

  tools {
    maven 'Maven 3.8.6'
  }

  environment {
    DOCKER_CREDENTIALS_ID = 'docker-hub'
    DOCKER_IMAGE = 'doniab7/springboot-with-docker'
    IMAGE_TAG = "${new Date().format('yyyyMMdd-HHmmss')}"
  }

  triggers {
    pollSCM('* * * * *') // Vérifie toutes les minutes (à adapter)
  }

  options {
    skipDefaultCheckout() // On va gérer le checkout manuellement
  }

  stages {

    stage('Cloner le dépôt') {
      steps {
        checkout([
          $class: 'GitSCM',
          branches: [[name: '*/main']],
          userRemoteConfigs: [[url: 'https://github.com/doniab7/CI-CD-pipeline-with-jenkins']]
        ])
      }
    }

    stage('Build Maven') {
      steps {
        sh 'mvn clean install -DskipTests=false' // Change à true si tu veux skip les tests
      }
    }

    stage('Exécuter les tests') {
      when {
        expression { fileExists('src/test') }
      }
      steps {
        sh 'mvn test'
      }
    }

    stage('Construire l’image Docker') {
      steps {
          sh "docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} ."
        }
    }

    stage('Pousser l’image sur Docker Hub') {
       steps {
          withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS_ID}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
            sh """
              echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
              docker push ${DOCKER_IMAGE}:${IMAGE_TAG}
            """
          }
        }
    }
  }

  post {
    success {
      echo "Pipeline exécuté avec succès. Image publiée : ${DOCKER_IMAGE}:${IMAGE_TAG}"
    }
    failure {
      echo "Le pipeline a échoué."
    }
  }
}
