pipeline {
  agent {
    docker {
      image 'squidfunk/mkdocs-material'
      args '-v ${PWD}:/docs build'
    }

  }
  stages {
    stage('build') {
      steps {
        archiveArtifacts 'site'
      }
    }

  }
}