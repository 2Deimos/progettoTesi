pipeline {
  agent any

  stages {

    stage("Build") {
      steps {
          bat 'mvn clean install -DskipTests'
        }
    }
    
        stage("Test") {
      steps {
          bat 'mvn test'
        }
    }

    stage ('Analisi OWASP Dependency Check') {
      steps {
        dependencyCheck additionalArguments: '''
        --project 'demoAppTesi'
        --scan target/*.jar
        --out target/
        --format ALL
        --prettyPrint
        --suppression owasp-suppressions.xml
        ''', odcInstallation: 'dependency-check-tool'
        
        dependencyCheckPublisher pattern: 'target/dependency-check-report.xml',
        stopBuild: true
      }
    }

    stage("Analisi SonarQube") {
      steps {
        withSonarQubeEnv('sonarqube-server') {
          bat 'mvn sonar:sonar'
          }
        }
    }

    stage("Quality Gate") {
      steps {
        timeout(time: 1, unit: 'HOURS') {
          waitForQualityGate abortPipeline: true
        }
      }
    }

    stage("Build Docker Image") {
      steps {
        bat 'docker build -t 2deimos/demoapptesi:build-%BUILD_NUMBER% .'
      }
    }

    stage("Push Docker Image") {
      steps {
        withCredentials([usernamePassword(credentialsId: 'DockerCredentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USER')]) {
          bat 'docker login --username %DOCKER_USER% --password %DOCKER_PASSWORD%'
          bat 'docker push 2deimos/demoapptesi:build-%BUILD_NUMBER%'
        }
      }
    }
  }
}
