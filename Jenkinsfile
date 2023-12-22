def COLOR_MAP = [
  'SUCCESS': 'good',
  'FAILURE': 'danger',
]

pipeline {
  agent any
  
  tools {
    maven "MAVEN3"
    jdk "OracleJDK8"
  }

  environment {
    SNAP_REPO = 'vprofile-snapshot'
    NEXUS_USER = 'admin'
    NEXUS_PASS = 'admin123'
    RELEASE_REPO = 'vprofile-release'
    CENTRAL_REPO = 'vpro-maven-central'
    NEXUSIP = '172.31.58.243'
    NEXUSPORT = '8081'
    NEXUS_GRP_REPO = 'vprofile-maven-group'
    NEXUS_LOGIN = 'nexuslogin'
    SONARSERVER = 'sonarserver'
    SONARSCANNER = 'sonarscanner'
  }

  stages {
    stage ('Build') {
      steps {
        sh 'mvn -s settings.xml -DskipTests install'
      }
      post {
        success {
          echo "Now Archiving."
          archiveArtifacts artifacts: '**/*.war'
        }
      }
    }

    stage ('Test') {
      steps {
        sh 'mvn -s settings.xml test'
      }
    }

    stage ('Checkstyle analysis') {
      steps {
        sh 'mvn -s settings.xml checkstyle:checkstyle'
      }
    }
  }
}