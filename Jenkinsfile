pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
	stage('Build Docker Image'){
	    when{
		branch 'master'
	    }
	    steps{
		script{
		    app = docker.build("crixguy/train-schedule")
		    app.inside {
			sh 'echo $(sleep 3; curl localhost:8080)'
		    }
		}
	    }
	}
	stage('Push Docker Image'){
	   when{ branch 'master'}
	   steps {
		script{
		    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login'){
			    app.push("${env.BUILD_NUMBER}")
			app.push("latest")
		    }
		}
	   } 
	}
	stage('Deploy To Production'){
	    when{ branch 'master'}
	    steps{
		input 'Deploy to Production?'
		milestone(1)
		withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]){
			script {
				sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_server \"docker pull crixguy/train-schedule:${env.BUILD_NUMBER}\""
		    		try{
					sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_server \"docker stop train-schedule\""
					sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_server \"docker rm train-schedule\""
		    		} catch (err){
					echo: 'caught error: $err'
				    }
			    	sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_server \"docker run --restart always --name train-schedule -p 8080:8080 -d crixguy/train-schedule:${env.BUILD_NUMBER}\""
			}
		}
	    }	    
	}
    }
}
