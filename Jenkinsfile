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
          sh "sudo mvn sonar:sonar -Dsonar.projectKey=demoprjt -Dsonar.host.url=http://192.168.223.20:9000 -Dsonar.login=29d0583597f0f60e8d91f7f56d49019955b5fc3d"
      }
    }
    stage('Vulnerability Scan - Docker ') {
      steps {
        sh "sudo mvn dependency-check:check"
      }
      post {
        always {
          dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
        }
      }
    }
    stage('Docker Build an Push') {
      steps {
        withDockerRegistry([credentialsId: "docker_hub", url: ""]) {
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
        withKubeConfig([credentialsId: 'kubeconfig']) {
          sh "sed -i 's#replace#mahendra770/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
          sh "sudo kubectl apply -f k8s_deployment_service.yaml"
        }
      }
    }
    stage('Email notification'){
      steps{
        mail bcc: '', body: 'jenkins job ,email alert', cc: '', from: '', replyTo: '', subject: 'jenkins job', to: 'polammahendra@gmail.com'
      }
    }
  }
}
