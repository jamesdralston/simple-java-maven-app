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
          sh 'mvn -q help:effective-settings -s /home/jenkins/.m2/settings.xml'
          sh 'mvn -V -B -e clean verify -Dmaven.test.failure.ignore  -s /home/jenkins/.m2/settings.xml'
        }
      }
      post {
        always {
          publishCoverage adapters: [jacoco(path: '**/*/jacoco.xml', thresholds: [[failUnhealthy: true, thresholdTarget: 'Line', unhealthyThreshold: 60.0]])], sourceFileResolver: sourceFiles('NEVER_STORE')
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
           sh 'mvn test -s /home/jenkins/.m2/settings.xml'
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
