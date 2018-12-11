pipeline{
	environment{
		scannerHome = tool 'Scanner';
		slackMet = load("slackNotifications.groovy");
		qg = waitForQualityGate();
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
			withSonarQubeEnv('SonarServer') {
			  bat "\"${scannerHome}/bin/sonar-scanner\""
			}
		}
	  }
	  stage("SonarQube Quality Gate") { 
		steps{
			timeout(time: 2, unit: 'MINUTES') {  
			   if(qg.status == "ERROR"){
				echo "Failed Quality Gates";
				slackMet.afterQG(qg.status);
				error "Pipeline aborted due to quality gate failure: ${qg.status}"
				waitForQualityGate abortPipeline: true
			   }
			   if (qg.status == 'OK') {
				 echo "Passed Quality Gates!";
				 slackMet.afterQG(qg.status);
			   }
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
			slackMet.call(currentBuild.currentResult);
		}
	  }
	  stage("Check App Status"){
		steps{
			echo "Checking if the App is live..."
			try{
				bat "curl -s --head  --request GET https://node-softinsa-app.eu-gb.mybluemix.net/ | grep '200 OK'"
				echo "The app is up and running!"
				slackMet.isRunning("Running");
			}catch(e){
				echo "The app is down..."
				slackMet.isRunning("NotRunning");
			}
		}
	  }
	}
}