node {
   def mvnHome
   stage('Preparation') { // for display purposes
      // Get some code from a GitHub repository
      git 'https://github.com/jjdiazl/simple-maven-project-with-tests.git'
      // Get the Maven tool.
      // ** NOTE: This 'M3' Maven tool must be configured
      // **       in the global configuration.           
      mvnHome = tool 'M3'
   }
/*   stage("UnitTest Source") {
  try {
    sh "'${mvnHome}/bin/mvn' clean test"
  } catch (err) {
   sh "exit -1"
  }
 }*/
 /*
  stage("Compile Source") {
  try {
    sh "'${mvnHome}/bin/mvn' clean compile package"
  } catch (err) {
   sh "exit -1"
  }
 }
 */
 
   stage('Build') {
      // Run the maven build
      if (isUnix()) {
         sh "'${mvnHome}/bin/mvn' -Dmaven.test.failure.ignore clean package"
      } else {
         bat(/"${mvnHome}\bin\mvn" -Dmaven.test.failure.ignore clean package/)
      }
   }
  
  
 stage("Package") {
  try {
      sh "echo package jjdl"
   sh "'${mvnHome}/bin/mvn' -Dmaven.test.failure.ignore install "
  } catch (err) {
   sh "exit -1"
  }
 }
	
  
 stage("CoverageTest Source") {
  try {
      sh "echo sonar report jjdl"
    sh "'${mvnHome}/bin/mvn' -Dmaven.test.failure.ignore scoverage:report sonar:sonar"
  } catch (err) {
   sh "exit -1"
  }
 }
   
   
   
 stage("Publish Artifact Nexus") {
  try {
      
    sh "'${mvnHome}/bin/mvn' -Dmaven.test.failure.ignore deploy"
   
  } catch (err) {
   sh "exit -1"
  }
 }
   
   stage('Results') {
      junit '**/target/surefire-reports/TEST-*.xml'
      archive 'target/*.jar'
   }
}
