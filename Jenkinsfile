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
						docker build -t adelriob/capstoneudacity .
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
						docker login -u sopotropo --password-stdin lltyrell
						docker push adelriob/capstoneudacity
					'''
				}
			}
		}
		stage('Set current kubectl context') {
			steps {
				withAWS(region:'us-east-2', credentials:'AKIAYZZNXNNA6PCTBTOL') {
					sh '''
						whoami
						kubectl version --short --client
						kubectl config use-context arn:aws:eks:us-east-2:793577097278:cluster/capstonecluster
					'''
				}
			}
		}
		stage('Deploy blue container') {
			steps {
				withAWS(region:'us-east-2', credentials:'AKIAYZZNXNNA6PCTBTOL') {
					sh '''
						kubectl apply -f ./blue_ctl.yml
					'''
				}
			}
		}
		stage('Deploy green container') {
			steps {
				withAWS(region:'us-east-2', credentials:'AKIAYZZNXNNA6PCTBTOL') {
					sh '''
						kubectl apply -f ./green_ctl.json
					'''
				}
			}
		}
		stage('Create the service in the cluster, redirect to blue') {
			steps {
				withAWS(region:'us-east-2', credentials:'AKIAYZZNXNNA6PCTBTOL') {
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
				withAWS(region:'us-east-2', credentials:'AKIAYZZNXNNA6PCTBTOL') {
					sh '''
						kubectl apply -f ./green_svc.yml
					'''
				}
			}
		}
	}
}
