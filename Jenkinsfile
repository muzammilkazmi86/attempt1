pipeline {
    agent any
    stages {
        stage('Lint HTML') {
            steps {
		sh 'echo "Performing lint check"'
                sh 'tidy -q -e *.html'
            }
        }
	    
	stage('Build Docker Image') {
   	    steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]){
		    sh 'echo "Deploying Docker Image..."'
     	    	    sh 'docker build -t muzammilkazmi86/capstoneattmp .'
		}
            }
        }
	    
	stage('Push Image To Dockerhub') {
   	    steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]){
		    sh 'echo "Image push for Docker Image..."'
     	    	    sh '''
                        docker login -u $USERNAME -p $PASSWORD
			docker push muzammilkazmi86/capstoneattmp
                    '''
		}
            }
        }
	    
	stage('Create kubernetes cluster') {
	    steps {
		withAWS(credentials: 'aws-kubectl', region: 'us-east-2') {
		    sh 'echo "Create kubernetes cluster..."'
		    sh '''
			eksctl create cluster \
			  --version 1.14 \
			  --region us-west-2 \
			  --node-type t3.medium \
			  --nodes 3 \
			  --nodes-min 1 \
			  --nodes-max 4 \
			  --name my-demo-cluster \
			  --kubeconfig=$HOME/kubeconfigs/demo-cluster-config.yaml
		'''
		}
	    }
        }
	    
	stage('Configure kubectl') {
	    steps {
		withAWS(credentials: 'aws-kubectl', region: 'us-east-2') {
		    sh 'echo "Configure kubectl..."'
		    sh 'aws eks --region us-east-2 update-kubeconfig --name mcu' 
		}
	    }
        }

	stage('Deploy blue container') {
	    steps {
		withAWS(credentials: 'aws-kubectl', region: 'us-east-2') {
		    sh 'echo "Deploy blue container..."'
		    sh 'kubectl apply -f blue.yaml'
		}
	    }
	}
	    
	stage('Deploy green container') {
	    steps {
		withAWS(credentials: 'aws-kubectl', region: 'us-east-2') {
		    sh 'echo "Deploy green container..."'
		    sh 'kubectl apply -f green.yaml'
		}
	    }
	}
	    
	stage('Create blue service') {
	    steps {
		withAWS(credentials: 'aws-kubectl', region: 'us-east-2') {
		    sh 'echo "Create blue service..."'
		    sh 'kubectl apply -f blue_service.yaml'
		}
	    }
	}
	    
	stage('Update service to green') {
	    steps {
		withAWS(credentials: 'aws-kubectl', region: 'us-east-2') {
		    sh 'echo "Update service to green..."'
		    sh 'kubectl apply -f green_service.yaml'
		}
	    }
	}	
   }   
	    
}
