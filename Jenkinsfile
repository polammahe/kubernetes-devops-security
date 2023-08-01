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
      post {
        always {
          junit 'target/surefire-reports/*.xml'
          jacoco execPattern: 'target/jacoco.exec'
        }
      }
    }
    stage('SonarQube - SAS') {
      steps {
          sh "sudo  mvn sonar:sonar -Dsonar.projectKey=secproject  -Dsonar.host.url=http://192.168.223.20:9000 -Dsonar.login=8432c679e41df19591c8c8fb00a933fc93013e42"
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
    stage('K8S CIS Benchmark') {
      steps {
        script {

          parallel(
            "Master": {
              sh "bash cis-master.sh"
            },
            "Etcd": {
              sh "bash cis-etcd.sh"
            },
            "Kubelet": {
              sh "bash cis-kubelet.sh"
            }
          )

        }
      }
    }
     stage('OWASP ZAP - DAST') {
      steps {
        withKubeConfig([credentialsId: 'kubeconfig']) {
          sh 'sudo bash zap.sh'
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
