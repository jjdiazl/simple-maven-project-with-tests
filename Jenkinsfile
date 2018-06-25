#!groovy

pipeline {
  agent any
  tools {
    maven 'M3'
  }
  options {
        //Si en 3 días no ha terminado que falle.
        timeout(time: 2, unit: 'HOURS') 
  }
  environment {
        //variable con el nombre del proyecto
        APP_NAME = 'My-Java-App'
  }
  
  stages {
    
          stage ('Initialize') { 
          //Primer paso, notificar inicio workflow
                   steps {
		   	sh 'echo configurar slack + hipchatsend'
			//slackSend (message: 'Inicio ejecucion ' + APP_NAME, channel: '#jenkins', color: '#0000FF', teamDomain: 'my-company', token: 'XXXXXXXXXXXXXXXXXXX' )
			//hipchatSend (color: 'GRAY', failOnError: true, notify: true, message: 'Inicio ejecucion ' + APP_NAME + ' <a href="${BLUE_OCEAN_URL}">Enlace a la ejecuci\u00F3n</a>', v2enabled: true,  room: 'Jenkins' )
                  }
	  }

          stage('Preparation') { // for display purposes
                   steps {
                      // Get some code from a GitHub repository
                      git 'https://github.com/jjdiazl/simple-maven-project-with-tests.git'
                   }
          }
	
          stage('Build') {//Fase de build
		  steps {
			  sh 'mvn -B -DskipTests clean package -Dmaven.test.failure.ignore clean package -Dmaven.test.skip=true'
                  }
          }
	  
	  stage ('Test') { //Fase de tests. En paralelo tests automaticos y de rendimiento
		  steps {
			  parallel 'Integration & Unit Tests': {
				  sh 'mvn test'
			}, 'Performance Test': {
				  sh 'mvn jmeter:jmeter'
			}
		  }
	  }
	
	  
	  
	  
	  
	  
	  
	  
	  
  }
}
