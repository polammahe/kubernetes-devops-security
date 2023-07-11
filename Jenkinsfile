pipeline {
  agent any
  stages {
    stage('Build Artifact - Maven') {
      steps {
        sh "sudo mvn clean package -DskipTests=true"
        archive 'target/*.jar'
      }
    }
    
    stage('Unit Test - JUnit and JaCoCo') {
      steps {
        sh "sudo mvn test"
      }
    }
    stage('SonarQube - SAST') {
      steps {
          sh "mvn sonar:sonar -Dsonar.projectKey=demoapp1 -Dsonar.host.url=http://innominds.eastasia.cloudapp.azure.com:9000 -Dsonar.login=3a86e02b3668309fc007b149869a5666de0aa02a"
      }
    }
    stage('Vulnerability Scan - Docker ') {
      steps {
        sh "mvn dependency-check:check"
      }
      post {
        always {
          dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
        }
      }
    }
    stage('Docker Build an Push') {
      steps {
        withDockerRegistry([credentialsId: "docker", url: ""]) {
          sh 'printenv'
          sh 'docker build -t mahendra770/numeric-app:""$GIT_COMMIT"" .'
          sh 'docker push mahendra770/numeric-app:""$GIT_COMMIT""'
        }
      }
    }
    stage("kube-scan"){
      steps{
        sh "bash kubesec-scan.sh"
      }
    }
    stage('Kubernete Deployment - DEV') {
      steps {
        withKubeConfig([credentialsId: 'kube-config']) {
          sh "sed -i 's#replace#mahendra770/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
          sh "kubectl apply -f k8s_deployment_service.yaml"
        }
      }
    }
  }
}
