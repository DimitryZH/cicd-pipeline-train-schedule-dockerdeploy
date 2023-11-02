pipeline {
	    agent any
	    stages {
	        stage('Build') {
	            steps {
	                echo 'Running build automation'
                    sh './gradlew build --no-daemon'
	                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
	            }
	        }
            stage('Build Docker Image') {
	            when {
	                branch 'master'
	            }
	            steps {
	                script {
	                    app = docker.build("dmitryzhuravlev/train-schedule-docker-deploy")

			
	                    app.inside {
	                        sh 'echo $(curl localhost:8080)'
	                    }
	                }
	            }
                }
		    stage('Push Docker Image') {
	            when {
	                branch 'master'
	            }
	            steps {
	                script {
	                    docker.withRegistry('https://index.docker.io/v1/', 'docker_hub_login') {
	                        app.push("${env.BUILD_NUMBER}")
	                        app.push("latest")
	                    }
	                }
	            }
	        }
	
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    script {
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker pull dmitryzhuravlev/train-schedule-docker-deploy:${env.BUILD_NUMBER}\""
                        try {
                            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker stop train-schedule-docker-deploy\""
                            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker rm train-schedule-docker-deploy\""
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker run --restart always --name train-schedule-docker-deploy -p 8080:8080 -d dmitryzhuravlev/train-schedule-docker-deploy:${env.BUILD_NUMBER}\""
                    }
                }
            }
        }
    }  
}
