pipeline {
  agent none
  stages {
    stage ('Build') {
      agent { 
        kubernetes {
          label 'maven-alpine-pod'
          yamlFile 'mvn-pod.yaml'
        }
      }  
      steps {
        container ('maven') {
          sh 'mvn -B -DskipTests clean package'
        }
      }
    }
  }
}
