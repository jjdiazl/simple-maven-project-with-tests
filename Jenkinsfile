#!groovy

def mvnHome = tool 'M3'

pipeline {
   	agent any    //Agente de Docker, de momento no utilizo Docker
     tools { //Alias a herramientas instaladas en Jenkins
        maven 'M3' //M3 es el nombre que le puse al maven instalado para Jenkins
        jdk 'JDK8' //JDK8 es el nombre que le puse al java de Jenkins
    }
    options {
        //Si en 3 días no ha terminado que falle.
        timeout(time: 76, unit: 'HOURS') 
    }
	
    environment {
        //variable con el nombre del proyecto
        APP_NAME = 'My-Java-App'
	//mvnHome = tool 'M3'
	
    }
	
    stages { //Inicio fases del workflow	
/*	stage ('Initialize') { 
		//Primer paso, notificar inicio workflow
             steps {
              //    slackSend (message: 'Inicio ejecucion ' + APP_NAME, channel: '#jenkins', color: '#0000FF', teamDomain: 'my-company', token: 'XXXXXXXXXXXXXXXXXXX' )
              //    hipchatSend (color: 'GRAY', failOnError: true, notify: true, message: 'Inicio ejecucion ' + APP_NAME + ' <a href="${BLUE_OCEAN_URL}">Enlace a la ejecuci\u00F3n</a>', v2enabled: true,  room: 'Jenkins' )
            }
        }
*/	
	stage('Preparation') { // for display purposes
             steps {
	      	// Get some code from a GitHub repository
      		git 'https://github.com/jjdiazl/simple-maven-project-with-tests.git'
      		// Get the Maven tool.
      		// ** NOTE: This 'M3' Maven tool must be configured
      		// **       in the global configuration.     
   		}
	}
	
	stage('Build') { // Build
      	     steps {
		// Run the maven build
		     echo mvnHome
                sh "'${mvnHome}/bin/mvn' -Dmaven.test.failure.ignore clean package -Dmaven.test.skip=true"
       		/*if (isUnix()) {
         	sh "'${mvnHome}/bin/mvn' -Dmaven.test.failure.ignore clean package"
      		} else {
         	bat(/"${mvnHome}\bin\mvn" -Dmaven.test.failure.ignore clean package/)*/
      		}
	}
	
	stage ('Test') { //Fase de tests. En paralelo tests automaticos y de rendimiento
            steps {
                 parallel 'Integration & Unit Tests': {
                     sh "'${mvnHome}/bin/mvn' test"
                 }, 'Performance Test': {
                     sh "'${mvnHome}/bin/mvn' jmeter:jmeter"
                 }
           }
        }
	
	stage ('QA') { //Fase de QA. En paralelo Sonar, Cobertura y OWASP
	    steps {
                parallel 'Sonarqube Analysis': {
                    //Si quieres ver la cobertura en sonar es necesario ejecutar cobertura y después sonar
                    sh "'${mvnHome}/bin/mvn' org.jacoco:jacoco-maven-plugin:prepare-agent install -Dmaven.test.failure.ignore=true"
                    sh "'${mvnHome}/bin/mvn' sonar:sonar"
                    echo 'Sonarqube Analysis'
               }, 'Cobertura code coverage' : {
                    //Realizamos análisis de cobertura de código
                    //Si la cobertura de código es inferior al 80% falla la ejecución y falla el workflow
                    sh "'${mvnHome}/bin/mvn' verify"
               }, 'OWASP Analysis' : {
                    sh "'${mvnHome}/bin/mvn' dependency-check:check"
               }
           }
           //Tras ejecutar los pasos en paralelos guardo el reporte de tests
           post {
               success {
                    junit 'target/surefire-reports/**/*.xml' 
               }
           }
        }
	
	stage("Publish Artifact Nexus") { //publicamos en nexus
        	steps {
			sh "'${mvnHome}/bin/mvn' -Dmaven.test.failure.ignore deploy"
  		}
	}
 	
	stage ('Deploy to Pre-production environment') {      	   //Desplegamos en el entorno de Pre-Producción
           //Se despliega en un tomcat con el plugin Cargo
           steps {
                sh "'${mvnHome}/bin/mvn' clean package cargo:redeploy -Dmaven.test.skip=true"
           }
        }
/*	 stage ('Confirmation') {
           //En esta fase esperamos hasta que la persona configurada confirme que desea subir a Producción. 
           //Tiene 72 horas para confirmar la subida a Producción.
           //Se envían notificaciones para que la persona tenga constancia
           steps {
                slackSend channel: '@dromeroa',color: '#00FF00', message: '\u00BFDeseas subir a produccion?. \n Confirma en la siguiente web: ${BLUE_OCEAN_URL}' , teamDomain: 'my-company', token: 'XXXXXXXXXXX'
                hipchatSend (color: 'YELLOW', failOnError: true, notify: true, message: '\u00BFDeseas subir a producci\u00F3n\u003F. \n Confirma en el siguiente <a href="${BLUE_OCEAN_URL}">Enlace</a>', textFormat: true, v2enabled: true, room: 'Jenkins')
                timeout(time: 72, unit: 'HOURS') {
                    input '\u00BFContinuar con despliegue en producci\u00F3n\u003F'
                }
           }
      }
      */
		
	stage ('Tagging the release candidate') {           //Realizamos un tag en GIT del código fuente
           steps {               //Tagging from trunk to tag
               echo "Tagging the release Candidate";
               sh "'${mvnHome}/bin/mvn' scm:tag -Dmaven.test.skip=true"
          }
       }
/*	stage ('Deploy to Production environment') {
           //Comenzamos a subir a producción en los dos servidores 
           steps {
                parallel 'Server 1': {
                    //Necesitamos realizar reintentos ya que falla la subida en remoto y se producen colisiones
                    retry(6) {
                        sh "'${mvnHome}/bin/mvn' tomcat7:redeploy -Dmaven.test.skip=true"
                    }
                }, 'Server 2' : {
                    retry(6) {
                        sh "'${mvnHome}/bin/mvn' tomcat:redeploy -Dmaven.test.skip=true"
                    }
                }
           }
      }
*/	
		
	stage ('CleanUp') {           //Limpiamos el workspace para no llenar los discos
           steps {
                deleteDir()
           }
        }
    }
}
	//Inicio de acciones post ejecución del workflow
    	//Notificamos como ha sido la ejecución del workflow
    	post {
      		success {
           		slackSend channel: '#jenkins',color: '#00FF00', message: APP_NAME + ' ejecutado satisfactoriamente.', teamDomain: 'my-company', token: 'XXXXXXXXXXXXXXXXXXXX'
           		hipchatSend (color: 'GREEN', failOnError: true, notify: true, message: APP_NAME + ' ejecutado satisfactoriamente. <a href="${BLUE_OCEAN_URL}">Enlace a la ejecuci\u00F3n</a>', textFormat: true, v2enabled: true, room: 'Jenkins')
      		}
      		failure {
           		slackSend channel: '#jenkins',color: '#FF0000', message: APP_NAME + ' se encuentra en estado fallido. ${BLUE_OCEAN_URL}', teamDomain: 'my-company', token: 'XXXXXXXXXXXXXXXX'
           		hipchatSend (color: 'RED', failOnError: true, notify: true, message: APP_NAME + ' se encuentra en estado fallido. <a href="${BLUE_OCEAN_URL}">Enlace a la ejecuci\u00F3n</a>', textFormat: true, v2enabled: true, room: 'Jenkins')
      		}
      	unstable {
           	slackSend channel: '#jenkins',color: '#FFFF00', message: APP_NAME + ' se encuentra en estado inestable. ${BLUE_OCEAN_URL}', teamDomain: 'my-company', token: 'XXXXXXXXXXXXXXXXXXXX'
           	hipchatSend (color: 'RED', failOnError: true, notify: true, message: APP_NAME + ' se encuentra en estado inestable. <a href="${BLUE_OCEAN_URL}">Enlace a la ejecuci\u00F3n</a>', textFormat: true, v2enabled: true, room: 'Jenkins')
      	}
}



	
