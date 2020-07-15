pipeline {
  agent {
    docker {
      image 'squidfunk/mkdocs-material build'
      args '-v ${PWD}:/docs'
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