pipeline {
   agent any

   stages {
	  stage('CompileandPackage') {
         steps {
			    git 'https://gitlab.com/vishnukiranreddy4/myproject.git'
			    sh label: '', script: 'mvn clean package'
			    echo "${BUILD_URL}"
         }
      }
	  stage('CodeAnalysis') {
         steps {
            script {
               def scannerHome = tool 'SonarScanner';
                  withSonarQubeEnv("SonarCloudServer") {
                     sh "${tool("SonarScanner")}/bin/sonar-scanner"
                  }
            }
        }
    }
	  stage('DeploytoTomcat') {
         steps {
			sh label: '', script: 'cp $(pwd)/target/*.war /opt/tomcat/webapps/'
			echo "${BUILD_URL}"
         }
      }
	  stage('FunctionalTesting') {
         steps {
		    sleep 60
			sh label: '', script: 'mvn -Dfilename=testng-functional.xml surefire:test'
         }
      }
   }
}