pipeline {
   agent any
 
   tools {
       maven 'maven3'
       jdk 'jdk17'
   }
 
  environment {
      SCANNER_HOME= tool 'sonar-scanner'
  }
  stages {
      stage('Git Checkout') {
          steps {
              git branch: 'main', url:'https://github.com/Kalyani43/Ekart.git'
                }
     }
 
     stage('Compile') {
         steps {
              sh "mvn compile"
               }
     }
 
     stage('Unit Tests') {
        steps {
              sh "mvn test -DskipTests=true"
        }
     }
 
     stage('SonarQube Analysis') {
        steps {
               withSonarQubeEnv('sonar') {
                 sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=EKART -Dsonar.projectName=EKART \
                 -Dsonar.java.binaries=. '''
               } 
        }
     }
 
    stage('OWASP Dependency Check') {
       steps {
              dependencyCheck additionalArguments: ' --scan ./', odcInstallation: 'DC' dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
    }
 
   stage('Build') {
      steps {
            sh "mvn package -DskipTests=true"
      }
   }
 
   stage('Deploy To Nexus') {
      steps {
            withMaven(globalMavenSettingsConfig: 'global-maven', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
            sh "mvn deploy -DskipTests=true"
            }
      }
   }
 
  stage('Docker Build & tag Image') {
     steps {
           script {
                  withDockerRegistry(credentialsId: 'docker-cred', toolName:'docker') {
                  sh "docker build -t kalyani43/ekart:latest -f docker/Dockerfile ."
                  }
           }
           }
  }
 
  stage('Trivy Scan') {
     steps {
           sh "trivy image kalyani43/ekart:latest > trivy-report.txt "
     }
  }
 
  stage('Docker Push Image') {
     steps {
           script {
            withDockerRegistry(credentialsId: 'docker-cred', toolName:'docker') {
            sh "docker push kalyani43/ekart:latest"
              }
            }
           }
   }

 
 }
}
