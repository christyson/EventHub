pipeline {
  agent {
    docker {
      image 'maven:latest'
    }
  }
  stages {
    stage('Build') {
      steps {
        sh 'mvn -q package'
      }
    }
    stage('SCA SC') {
      steps {
        withCredentials([string(credentialsId: 'SRCCLR_API_TOKEN', variable: 'SRCCLR_API_TOKEN')]) {
          sh '''
            curl -sSL https://download.sourceclear.com/ci.sh | sh
          '''
        }
      }
    }
    stage('Veracode Sast') {
      // upload and scan
      steps {
        withCredentials([usernamePassword(credentialsId: 'Veracode', passwordVariable: 'VERACODEKEY', usernameVariable: 'VERACODEID')]) {
          veracode applicationName: '$JOB_NAME', canFailJob: false, createProfile: true, criticality: 'High', fileNamePattern: '', replacementPattern: '', sandboxName: '', scanExcludesPattern: '', scanIncludesPattern: '', scanName: "$BUILD_NUMBER", teams: '', timeout: 60, uploadExcludesPattern: '', uploadIncludesPattern: '**/**.war', useIDkey: true, vid: VERACODEID, vkey: VERACODEKEY, vpassword: '', vuser: ''
        }
      }
    }
    stage('Test') {
      steps {
        // 1.	Enable Veracode Interactive for the steps that run the tests. 
        wrap([$class: 'VeracodeInteractiveBuildWrapper', location: 'host.docker.internal', port: '10010']) {
          // 2.	Download the IAST Agent into the project workspace. 
          sh 'curl -sSL https://s3.us-east-2.amazonaws.com/app.veracode-iast.io/iast-ci.sh |  sh'
          // 3.	Run the tests with the Veracode Interactive Agent attached. 
          sh 'mvn -q -am -pl web clean test'
        }
      }
    }
    stage('Deploy') {
      steps {
        sh 'echo npm package would run here...'
      }
    }
  }
}
