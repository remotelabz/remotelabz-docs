pipeline {
  agent none
  stages {
    stage('build') {
      agent {
        docker {
          image 'squidfunk/mkdocs-material'
          args '--entrypoint=\'\''
        }

      }
      steps {
        sh 'mkdocs build'
      }
    }

    stage('publish') {
      agent {
        node {
          label 'local'
        }

      }
      environment {
        GITHUB_TOKEN = 'a049a1ab4ef65d3df588b68074c2dd32f9249810'
      }
      steps {
        sh '''sudo rm -rf /var/www/remotelabz-docs || true
sudo mv site /var/www/remotelabz-docs
sudo chmod -R 755 /var/www/remotelabz-docs
sudo chown -R www-data:www-data /var/www/remotelabz-docs'''
      }
    }

  }
}