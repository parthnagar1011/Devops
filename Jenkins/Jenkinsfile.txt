pipeline {
  agent { label 'agent1' }
  stages {
    stage('Environment') {
      steps {
        bat 'echo Running on %COMPUTERNAME% & whoami & cd'
      }
    }
    stage('Java Check') {
      steps {
        bat 'java -version'
      }
    }
  }
}
