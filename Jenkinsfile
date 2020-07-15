pipeline {
  agent {
    docker {
      image 'squidfunk/mkdocs-material'
    }
  }
  stages {
    stage('build') {
      steps {
        sh 'build'
        archiveArtifacts 'site'
      }
    }

  }
}
