pipeline {
	agent any

	environment {
		DOCKER_IMAGE = "thuongtx/spring-jenkins-pipleline"
	}

	stages {
		stage('Clone') {
			steps {
				git branch: 'main', url: 'https://github.com/thuongtrinh/spring-jenkins-pipleline.git'
			}
		}

		stage("Test") {
			steps {
				echo "Test Stage"
				//sh "mvn test"
				//step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/TEST-*.xml'])
				withMaven(maven : 'apache-maven-3.8.1') {
					bat 'mvn test'
				}
			}
		}

//		stage('Build') {
//			steps {
//				withDockerRegistry(credentialsId: 'docker-hub-webhook1', url: 'https://index.docker.io/v1/') {
//					sh 'docker build -t thuongtx/docker-spring-boot:v1 .'
//					sh 'docker push thuongtx/docker-spring-boot:v1'
//				}
//			}
//		}

		stage("Build") {
			agent {
				node {label 'main'}
			}
			environment {
				DOCKER_TAG="${GIT_BRANCH.tokenize('/').pop()}-${GIT_COMMIT.substring(0,7)}"
			}
			steps {
				sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} . "
				sh "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest"
				sh "docker image ls | grep ${DOCKER_IMAGE}"
				withCredentials([usernamePassword(credentialsId: 'docker-hub-webhook1', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
					sh 'echo $DOCKER_PASSWORD | docker login --username $DOCKER_USERNAME --password-stdin'
					sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
					sh "docker push ${DOCKER_IMAGE}:latest"
				}

				//clean to save disk
				sh "docker image rm ${DOCKER_IMAGE}:${DOCKER_TAG}"
				sh "docker image rm ${DOCKER_IMAGE}:latest"
			}
		}
	}

	post {
		success {
			echo "SUCCESSFUL"
		}
		failure {
			echo "FAILED"
		}
	}
}
