pipeline {
  agent {
    docker {
      image 'squidfunk/mkdocs-material'
      args '--entrypoint=\'\''
    }

  }
  stages {
    stage('build') {
      steps {
        sh 'pwd'
        archiveArtifacts 'site'
      }
    }

  }
}