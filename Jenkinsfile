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
						docker image list
						docker ps
					'''
				}
			}
		}
		stage('Push Image To Dockerhub') {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh '''
						docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
						docker push sopotropo/capstoneudacity
						docker image list
						docker ps
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
						docker image list
						docker ps
						kubectl get pods --all-namespaces -o wide
						kubectl delete pod blue-w7dx5
						kubectl delete pod green-9bwvw
						
					'''
				}
			}
		}
		stage('Deploy blue container') {
			steps {
				withAWS(region:'us-east-2', credentials:'aws-static') {
					sh '''
						kubectl apply -f ./blue_ctl.yml
						docker image list
						docker ps
						kubectl get pods --all-namespaces -o wide
					'''
				}
			}
		}
		stage('Deploy green container') {
			steps {
				withAWS(region:'us-east-2', credentials:'aws-static') {
					sh '''
						kubectl apply -f ./green_ctl.yml
						docker image list
						docker ps
						kubectl get pods --all-namespaces -o wide
					'''
				}
			}
		}
		stage('Create the service in the cluster, redirect to blue') {
			steps {
				withAWS(region:'us-east-2', credentials:'aws-static') {
					sh '''
						kubectl apply -f ./blue_svc.yml
						docker image list
						docker ps
						kubectl get pods --all-namespaces -o wide
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
						docker image list
						docker ps
						kubectl get pods --all-namespaces -o wide
					'''
				}
			}
		}
	}
}
