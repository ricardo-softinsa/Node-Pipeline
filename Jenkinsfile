pipeline{ 
	environment{
		scannerHome = tool 'Scanner';
		slackMet = load 'slackNotifications.groovy';
	}

	agent any
	
	stages{
	  stage('SCM') {
		steps{
			git 'https://github.com/ricardo-softinsa/Node-Pipeline.git'
		}
	  }
	  stage('SonarQube analysis') {
		steps{
			echo "SonarQube analysis"
			echo "${scannerHome}"
			withSonarQubeEnv('SonarServer') {
			  sh "\"${scannerHome}/bin/sonar-scanner\""
			}
			sleep 5
		}
	  }
	  stage("SonarQube Quality Gate") { 
		steps{
		    echo "SonaQube Quality Gate"
		    timeout(time: 1, unit: 'MINUTES') {  
				waitForQualityGate abortPipeline: true
		    }
		}
	  }
	  stage("Pushing to Cloud"){
		steps{
			echo "Pushing into the cloud...";
			cfPush(
				target: 'api.eu-gb.bluemix.net',
				organization: 'ricardo.miguel.magalhaes@pt.softinsa.com',
				cloudSpace: 'dev',
				credentialsId: 'CFPush',
			)
		}
	  }
	  stage("Check App Status"){
		steps{
			echo "Checking if the App is live..."
			script{
				try{
					sh "curl -s --head  --request GET https://node-softinsa-app.eu-gb.mybluemix.net/ | grep '200 OK'"
					echo "The app is up and running!"
					slackSend color: "good", message: "${env.JOB_NAME} #${env.BUILD_NUMBER} - Your app is up and running!"
				}catch(e){
					echo "The app is down..."
					slackSend color: "danger", message: "${env.JOB_NAME} #${env.BUILD_NUMBER} - Your app is down..."
				}
			}
		}
	  }
	}
	post{
		success{
			echo "something"
			slackSend color: "good", message: "${env.JOB_NAME} #${env.BUILD_NUMBER} was successful!"
		}
		unstable{
			echo "something"
			slackSend color: "#b300b3", message: "${env.JOB_NAME} #${env.BUILD_NUMBER} is unstable!"
		}
		failure{
			echo "something"
			slackSend color: "danger", message: "${env.JOB_NAME} #${env.BUILD_NUMBER} has failed..."
		}
		aborted{
			echo "something"
			slackSend color: "#000000", message: "${env.JOB_NAME} #${env.BUILD_NUMBER} has aborted..."
		}
		changed{
			echo "something"
			slackSend color: "#ff9900", message: "${env.JOB_NAME} #${env.BUILD_NUMBER} has changed since last build."
		}
	}
}
