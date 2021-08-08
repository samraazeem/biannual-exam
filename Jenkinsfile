pipeline{ 
    environment {
    registry = "samraazeem/nagp_assignment"
    registryCredential = 'docker'
    dockerImage= ''
    }  
    agent any 
    tools{
        maven 'MAVEN'
        npm 'NPM'
    }
    
    options {
        skipDefaultCheckout(true)
    }

    stages {
        stage("Code Checkout") {
            steps {
                git url: 'https://github.com/samraazeem/microservice-assignment.git'
            }
        }
        stage('Build') {
            steps{
                sh 'npm install'
                sh 'npm build --prod'  
            }
        }
        stage('Unit Testing'){
            when {
                branch 'development'
            }
            steps{
               sh 'ng test --codeCoverage=true --watcher=true'
            } 
        }
        stage('SonarQube Analysis'){
            when {
                branch 'production'
            }
			steps{
				withSonarQubeEnv('SONAR'){
					sh 'npm run sonar'
				}
			}
		}
        stage('Docker Image') { 
            steps{
                
                /***sh 'docker build -t "samraazeem/angular-carousel":$BUILD_NUMBER"" .'
                script {
                    dockerImage= 'samraazeem/angular-carousel":$BUILD_NUMBER"'
                }***/
                script {
                    dockerImage= docker.build registry + ":$BUILD_NUMBER"
                }
            }
        }
        stage('Container'){
            parallel {
                stage('PreContainer Check'){
                    steps{
                        sh 'docker rm -f angular-carousel'
                    }
                }
                stage('Publish DockerHub'){
                    steps{
                        script {
                            docker.withRegistry( '', registryCredential ) {
                            dockerImage.push()
                            }
                        }
                    }
                }
            }  
        }
        stage('Docker Deployment'){
            /***parallel {
                stage('Docker Deploy Development'){
                    steps{
                        sh 'docker rm -f angular-carousel'
                        sh 'docker run -d --name angular-carousel -p 7200:80 samraazeem/carousel-angular:"$BUILD_NUMBER"'
                    }
                }***/
               // stage('Docker Deploy Production'){
                    steps{
                        sh 'docker rm -f angular-carousel'
                        sh 'docker run -d --name angular-carousel -p 7300:80 samraazeem/carousel-angular:"$BUILD_NUMBER"'
                    }
               // }
            //}  
        } 
        stage('Kubernetes Deployment'){
            //parallel {
            //    stage('Docker Deploy Development'){
                    steps{
                        sh 'kubectl apply -f ./kubernetes/frontend.yaml'
                        sh 'kubectl apply -f ./kubernetes/backend.yaml'
                    }
              //  }
              /***  stage('Docker Deploy Production'){
                    steps{
                        sh 'docker run -d --name angular-carousel -p 7300:80 samraazeem/carousel-angular:"$BUILD_NUMBER"'
                    }
                }
            } ***/
        } 
    }
}