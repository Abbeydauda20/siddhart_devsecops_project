pipeline {
  agent any

  stages {
    stage('Build Artifact - Maven') {
      steps {
        sh "mvn clean package -DskipTests=true"
        archiveArtifacts artifacts: 'target/*.jar'
      }
    }

    stage('Unit Tests - JUnit and JaCoCo') {
      steps {
        sh "mvn test"
      }
      post {
        always {
          junit 'target/surefire-reports/*.xml'
          jacoco execPattern: 'target/jacoco.exec'
        }
      }
    }

    stage('Mutation Tests - PIT') {
      steps {
        sh "mvn org.pitest:pitest-maven:mutationCoverage"
      }
      post {
        always {
          pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
        }
      }
    }

    stage('SonarQube - SAST') {
      steps {
        withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
          sh """
            mvn clean verify sonar:sonar \
            -Dsonar.projectKey=DevSecOps_Siddhart \
            -Dsonar.projectName='DevSecOps_Siddhart' \
            -Dsonar.host.url=http://localhost:9000 \
            -Dsonar.token=${SONAR_TOKEN}
          """
        }
      }
    }

    stage('Docker Build and Push') {
      steps {
        withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
          sh 'printenv'
          sh 'docker build -t siddharth67/numeric-app:"${GIT_COMMIT}" .'
          sh 'docker push siddharth67/numeric-app:"${GIT_COMMIT}"'
        }
      }
    }

    stage('Kubernetes Deployment - DEV') {
      steps {
        withKubeConfig([credentialsId: 'kubeconfig']) {
          sh "sed -i 's#replace#siddharth67/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
          sh "kubectl apply -f k8s_deployment_service.yaml"
        }
      }
    }
  }
}
