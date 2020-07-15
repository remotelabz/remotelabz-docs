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
        sh 'mkdocs build'
        archiveArtifacts './site'
      }
    }

  }
}