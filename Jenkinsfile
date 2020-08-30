pipeline {
	agent any
	stages {
		stage('Lint') {
			steps {
				sh '''
					tidy -q -e *.html
				'''
			}
		}
		stage('Build Docker Image') {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh '''
						docker build -t sopotropo/capstoneudacity .
					'''
				}
			}
		}
		stage('Push Image To Dockerhub') {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh '''
					    echo $DOCKER_USERNAME
					    echo $DOCKER_PASSWORD
						docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
						docker push sopotropo/capstoneudacity
					'''
				}
			}
		}
		stage('Set current kubectl context') {
			steps {
				withAWS(region:'us-east-2', credentials:'aws-static') {
					sh '''
						whoami
						kubectl version --short --client
						kubectl config use-context arn:aws:eks:us-east-2:605149424449:cluster/capstonecluster
					'''
				}
			}
		}
		stage('Deploy blue container') {
			steps {
				withAWS(region:'us-east-2', credentials:'aws-static') {
					sh '''
						kubectl apply -f ./blue_ctl.yml
					'''
				}
			}
		}
		stage('Deploy green container') {
			steps {
				withAWS(region:'us-east-2', credentials:'aws-static') {
					sh '''
						kubectl apply -f ./green_ctl.yml
					'''
				}
			}
		}
		stage('Create the service in the cluster, redirect to blue') {
			steps {
				withAWS(region:'us-east-2', credentials:'aws-static') {
					sh '''
						kubectl apply -f ./blue_svc.yml
					'''
				}
			}
		}
		stage('Wait user approve') {
            steps {
                input "Ready to redirect traffic to green?"
            }
        }
		stage('Create the svc in Cluster, redirect to green') {
			steps {
				withAWS(region:'us-east-2', credentials:'aws-static') {
					sh '''
						kubectl apply -f ./green_svc.yml
					'''
				}
			}
		}
	}
}
