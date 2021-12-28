pipeline {
  agent any

  stages {
    stage('Build Artifact - Maven') {
      steps {
        sh "mvn clean package -DskipTests=true"
        archive 'target/*.jar'
      }
    }

    stage('Unit Tests - JUnit and Jacoco') {
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
	    stage('SonarQube - SAST') {
      steps {
        withSonarQubeEnv('SonarQube') {
          sh "mvn sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.host.url=http://devsecops-demo.eastus.cloudapp.azure.com:9000 -Dsonar.login=62bc740c3a4fe6fa92153c15df1903123a32bc98"
        }
        timeout(time: 2, unit: 'MINUTES') {
          script {
            waitForQualityGate abortPipeline: true
          }
        }
      }
    }
//	    stage('Vulnerability Scan - Docker ') {
  //    steps {
//        sh "mvn dependency-check:check"
//      }
//      post {
//        always {
//          dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
//        }
//      }
//    }
//	  stage('Vulnerability Scan - Docker') {
//      steps {
//        parallel(
 //         "Dependency Scan": {
//            sh "mvn dependency-check:check"
//          },
//          "Trivy Scan": {
//            sh "bash trivy-docker-image-scan.sh"
//          }
 //       )
//      }
//    }
	    stage('Docker Build and Push') {
      steps {
        withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
          sh 'printenv'
          sh 'docker build -t vrobins77/numeric-app:""$GIT_COMMIT"" .'
          sh 'docker push vrobins77/numeric-app:""$GIT_COMMIT""'
        }
      }
    }

	    stage('Kubernetes Deployment - DEV') {
      steps {
        withKubeConfig([credentialsId: '90b05547-147e-4181-ba0b-b7a287c7fd37']) {
          sh "sed -i 's#replace#vrobins77/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
          sh "kubectl apply -f k8s_deployment_service.yaml"
        }
      }
    }
  }
}
