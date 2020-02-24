pipeline {
  agent none
  stages {
    stage ('Run Maven') {
      agent { 
        kubernetes {
          label 'maven-alpine-pod'
          yamlFile 'mvn-pod.yaml'
        }
      }  
      steps {
        container ('maven') {
          echo 'Hello Maven!'
          sh 'mvn -version'
        }
      }
    }
  }
}
