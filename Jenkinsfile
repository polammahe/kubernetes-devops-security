pipeline {
  agent any
  stages {
    stage('Build Artifact - Maven') {
      steps {
        sh "mvn clean package -DskipTests=true"
        archive 'target/*.jar'
      }
    }
    
    stage('Unit Test - JUnit and JaCoCo') {
      steps {
        sh "mvn test"
      }
    }
    stage('SonarQube - SAST') {
      steps {
          sh "mvn sonar:sonar -Dsonar.projectKey=secproject1 -Dsonar.host.url=http://securitydemo.eastasia.cloudapp.azure.com:9000 -Dsonar.login=dd6df3873260206cfcd6ae7b952ed1020a3986e4"
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
    stage('Docker Build and Push') {
      steps {
        withDockerRegistry([credentialsId: "dockerhub", url: ""]) {
          sh 'printenv'
          sh 'docker build -t mahendra770/numeric-app:""$GIT_COMMIT"" .'
          sh 'docker push mahendra770/numeric-app:""$GIT_COMMIT""'
        }
      }
    }
    stage('Vulnerability Scan - Kubernetes') {
      steps {
        sh 'docker run --rm -v $(pwd):/project openpolicyagent/conftest test --policy opa-k8s-security.rego k8s_deployment_service.yaml'
      }
    }
    stage("kube-scan"){
      steps{
        sh "bash kubesec-scan.sh"
      }
    }
    stage('Kubernetes Deployment - DEV') {
      steps {
        withKubeConfig([credentialsId: 'kube-config']) {
          sh "sed -i 's#replace#mahendra770/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
          sh "kubectl apply -f k8s_deployment_service.yaml"
        }
      }
    }
  }
}
