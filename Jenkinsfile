pipeline {
  agent any
  tools { 
    maven 'Maven_3_8_4'  
  }

  stages {
    stage('CompileandRunSonarAnalysis') {
      steps {
        sh '''
          mvn clean verify sonar:sonar \
            -Dsonar.projectKey=udayvarma_varmabuggywebapp \
            -Dsonar.organization=udayvarma \
            -Dsonar.host.url=https://sonarcloud.io \
            -Dsonar.login=460a499e8d80d82171867e00ff438d6b13f420c2
        '''
      }
    }

    stage('RunSCAAnalysisUsingSnyk') {
      steps {
        withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
          sh 'mvn snyk:test -fn'
        }
      }
    }

    stage('Build') {
      steps {
        withDockerRegistry([credentialsId: "dockerlogin", url: ""]) {
          script {
            app = docker.build("udayvarma77")
          }
        }
      }
    }

    stage('Push') {
      steps {
        script {
          docker.withRegistry(
            'https://410129828206.dkr.ecr.us-west-2.amazonaws.com',
            'ecr:us-west-2:aws-credentials'
          ) {
            app.push("latest")
          }
        }
      }
    }
    stage('Kubernetes Deployment of ASG Bugg Web Application') {
	   steps {
	      withKubeConfig([credentialsId: 'kubelogin']) {
		  sh('kubectl delete all --all -n devsecops')
		  sh ('kubectl apply -f deployment.yaml --namespace=devsecops')
		}
	      }
   	}

  }
}

