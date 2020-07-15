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
        archiveArtifacts 'site/**/*'
      }
    }

    stage('upload') {
      environment {
        GITHUB_TOKEN = 'a049a1ab4ef65d3df588b68074c2dd32f9249810'
      }
      steps {
        sh '''echo "Publishing on Github..."
token=$GITHUB_TOKEN

tag=$(git describe --tags)

message="$(git for-each-ref refs/tags/$tag --format=\'%(contents)\')"

name=$(echo "$message" | head -n1)
description=$(echo "$message" | tail -n +3)
description=$(echo "$description" | sed -z \'s/\\n/\\\\n/g\') # Escape line breaks to prevent json parsing problems

release=$(curl -XPOST -H "Authorization:token $token" --data "{\\"tag_name\\": \\"$tag\\", \\"target_commitish\\": \\"master\\", \\"name\\": \\"$name\\", \\"body\\": \\"$description\\", \\"draft\\": false, \\"prerelease\\": true}" https://api.github.com/repos/crestic-urca/remotelabz-docs/releases)

id=$(echo "$release" | sed -n -e \'s/"id":\\ \\([0-9]\\+\\),/\\1/p\' | head -n 1 | sed \'s/[[:blank:]]//g\')

curl -XPOST -H "Authorization:token $token" -H "Content-Type:application/octet-stream" --data-binary @artifact.zip https://uploads.github.com/repos/crestic-urca/remotelabz-docs/releases/$id/assets?name=artifact.zip'''
      }
    }

  }
}