pipeline {
  agent { 
    kubernetes {
      label 'maven-alpine-pod'
      podRetention always()
      idleMinutes 120
      yamlFile 'mvn-pod.yaml'
     }
  }  
  stages {
    stage ('Build and Analysis') {
      steps {
        container ('maven') {
          sh 'mvn -V -q -e clean verify -Dmaven.test.failure.ignore'
        }
      }
      post {
        always {
          jacoco()
          recordIssues enabledForFailure: true,  tools: [java(), javaDoc()], aggregatingResults: 'true', id: 'java', name: 'Java'
          recordIssues enabledForFailure: true, tool: errorProne(), healthy: 1, unhealthy: 20
          recordIssues enabledForFailure: true, tools: [pmdParser(pattern: 'target/pmd.xml'),
            spotBugs(pattern: 'target/spotbugsXml.xml')],
            qualityGates: [[threshold: 1, type: 'TOTAL', unstable: true]]
          recordIssues enabledForFailure: true, tools: [checkStyle(pattern: 'target/checkstyle-result.xml'),
            cpd(pattern: 'target/cpd.xml')]
        }
      }
    }
    stage ('Test') {
      steps {
         container ('maven') {
           sh 'mvn -q test'
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
