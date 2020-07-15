pipeline {
  agent {
    docker {
      image 'squidfunk/mkdocs-material'
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