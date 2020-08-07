pipeline {
   environment {
    image = "219244194696.dkr.ecr.us-east-1.amazonaws.com/demoproject"
	version = "1.0.0.$BUILD_NUMBER"
	NAMESPACE= "demoproject"
	RELEASE_NAME = "demoproject"
	region= "us-east-1"
	cluster_name= "Cluster-bFxlemlV1odQ"
	
   }
  agent {
    node {
      label 'master'
    }

  }
	

  stages {
	  stage('Run Unit Testing') {
      steps {
        sh '''echo "Running Unit Testing"
	      mvn clean test'''
      }
    }
	  
    stage('Run Build') {
      steps {
        sh ''' echo "Compiling and building the maven project"
	      mvn clean package'''
      }
    }
    
	  
	  stage(' Run Code Quality Analysis using SonarQube') {
      steps {
        sh ''' echo "Running Sonar code quality Analysis"
	      mvn sonar:sonar -Dsonar.host.url=http://0.0.0.0:9000'''
      }
    }
	  
	  
    stage('Build Docker Image and Push to ECR') {
      steps {
        echo "Build docker image"
        script {
           sh '''docker build -t $image:$version .
	         echo "Login to ECR Repo"
		 $(aws ecr get-login --no-include-email --region us-east-1 | sed 's|https://||')
		 echo "Pushing docker image to ECR"
		 docker push $image:$version
		 docker rmi $image:$version
		 '''
            
           }

      }
    }
	
     stage('Approval for Deployment') {
       steps {
            script {
              timeout(time: 10, unit: 'MINUTES') {
                input(id: "Deploy", message: "Are you sure to deploy current release into target environment", ok: 'Deploy')
              }
            }
        }
    }
	
	stage('Deploy in K8s') {
       steps {
             sh """
	     echo "Update the context or kubeconfig"
	     aws eks --region $region update-kubeconfig --name $cluster_name
	     echo "Updating the version"
	     cd chart
	     sed -i 's/dockertag/$version/' values.yaml
	     helm install -n $NAMESPACE $RELEASE_NAME .
	     """
        }
    }
	  
  }
 /**   post {
    success {
      emailext (
          subject: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
          body: """<p>SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
            <p>Check console output;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a></p>""",
          to: 'rajeev.k9937@gmail.com'
        )
    }

    failure {
      
      emailext (
          subject: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
          body: """<p>FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
            <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
          to: 'rajeev.k9937@gmail.com'
        )
    }
  } **/
}
