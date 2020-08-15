pipeline {
  agent { 
    kubernetes {
      label 'maven-alpine-pod'
       yamlFile 'mvn-pod.yaml'
     }
  }  
  stages {
    stage ('Build and Analysis') {
      steps {
        container ('maven') {
          sh 'mvn -V -e clean verify -Dmaven.test.failure.ignore'
        }
      }
      post {
        always {
          recordIssues tools: [java(), javaDoc()], aggregatingResults: 'true', id: 'java', name: 'Java'
          recordIssues tool: errorProne(), healthy: 1, unhealthy: 20
          recordIssues tools: [checkStyle(pattern: 'target/checkstyle-result.xml'),
            spotBugs(pattern: 'target/spotbugsXml.xml'),
            pmdParser(pattern: 'target/pmd.xml'),
            cpd(pattern: 'target/cpd.xml')],
            qualityGates: [[threshold: 1, type: 'TOTAL', unstable: true]]
        }
      }
    }
    stage ('Test') {
      steps {
         container ('maven') {
           sh 'mvn test'
         }
      }
      post {
        always {
          junit 'target/surefire-reports/*.xml'
        }
      }
    }
    stage ('Deliver') {
      when {
        beforeAgent true
        branch 'master'
      }
      steps {
        container ('maven') {
          sh './jenkins/scripts/deliver.sh'
        }
      }
    }
  }
}
