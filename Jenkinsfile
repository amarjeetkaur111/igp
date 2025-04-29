pipeline {
    agent any

    environment {
        MAVEN_HOME = '/opt/maven/current'
		PATH = "${MAVEN_HOME}/bin:${env.PATH}"
		JOB_NAME = 'igp-cicd-pl'
		DOCKER_IMAGE = "amarjeetkaur111/igp-cicd"
    	DOCKER_TAG = "latest"
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Cloning repository..."
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo "Building the project with Maven..."
                sh 'mvn clean compile'
            }
        }

        stage('Test') {
            steps {
                echo "Running tests..."
                sh 'mvn test'
            }
        }

        stage('Package') {
            steps {
                echo "Packaging the application..."
                sh 'mvn package'
            }
        }
		stage('Docker Build & Push') 
		{
			steps {
				withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
				sh """
					cp /var/lib/jenkins/workspace/$JOB_NAME/target/ABCtechnologies-1.0.war target/abc.war	
					docker build -t $DOCKER_IMAGE:$DOCKER_TAG .
					docker push $DOCKER_IMAGE:$DOCKER_TAG
				"""
				}
			}
    	}
		stage('Deploy to Kubernetes') 
		{
            steps {
                sh 'kubectl apply -f k8s/deploy.yaml || kubectl replace -f k8s/deploy.yaml'
                sh 'kubectl apply -f k8s/svc.yaml || kubectl replace -f k8s/svc.yaml'
                sh 'kubectl apply -f k8s/prometheus.yaml || kubectl replace -f k8s/prometheus.yaml'
            }
        }

    }

    post {
        success {
            echo '✅ Build succeeded!'
        }
        failure {
            echo '❌ Build failed.'
        }
    }
}
