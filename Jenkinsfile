pipeline {
  agent {
    docker {
      image 'squidfunk/mkdocs-material'
      args '''
'''
    }

  }
  stages {
    stage('build') {
      steps {
        sh 'mkdocs build'
        archiveArtifacts 'site'
      }
    }

  }
}