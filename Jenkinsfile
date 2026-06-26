pipeline {
  agent {
    kubernetes {
      label 'kaniko-agent'
      defaultContainer 'git'
      yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
    - name: git
      image: alpine/git:2.45.2
      command:
        - cat
      tty: true

    - name: kaniko
      image: gcr.io/kaniko-project/executor:debug
      command:
        - /busybox/cat
      tty: true
"""
    }
  }

  environment {
    DOCKERHUB_USER = 'hamidhosseini2'
    APP_NAME = 'my-app'
    MANIFEST_REPO = 'github.com/Hamid-it/my-app-manifests.git'
  }

  stages {
    stage('Checkout App Repo') {
      steps {
        checkout scm
      }
    }

    stage('Set Image Tag') {
      steps {
        script {
          sh 'git config --global --add safe.directory "$WORKSPACE"'

          def shortCommit = sh(
            script: 'git rev-parse --short HEAD',
            returnStdout: true
          ).trim()

          env.IMAGE_TAG = "${env.BUILD_NUMBER}-${shortCommit}"
          env.IMAGE = "${env.DOCKERHUB_USER}/${env.APP_NAME}:${env.IMAGE_TAG}"

          echo "Image will be: ${env.IMAGE}"
        }
      }
    }

    stage('Build and Push Image') {
      steps {
        container('kaniko') {
          withCredentials([
            usernamePassword(
              credentialsId: 'dockerhub-creds',
              usernameVariable: 'DOCKER_USER',
              passwordVariable: 'DOCKER_PASS'
            )
          ]) {
            sh '''
              mkdir -p /kaniko/.docker

              AUTH=$(printf "%s:%s" "$DOCKER_USER" "$DOCKER_PASS" | base64 | tr -d '\\n')

              cat > /kaniko/.docker/config.json <<EOF
{
  "auths": {
    "https://index.docker.io/v1/": {
      "auth": "$AUTH"
    }
  }
}
EOF

              /kaniko/executor \
                --context "$WORKSPACE" \
                --dockerfile "$WORKSPACE/Dockerfile" \
                --destination "$IMAGE"
            '''
          }
        }
      }
    }

    stage('Update Manifest Repo') {
      steps {
        container('git') {
          withCredentials([
            usernamePassword(
              credentialsId: 'github-creds',
              usernameVariable: 'GITHUB_USER',
              passwordVariable: 'GITHUB_TOKEN'
            )
          ]) {
            sh '''
              rm -rf my-app-manifests

              git clone https://${GITHUB_USER}:${GITHUB_TOKEN}@${MANIFEST_REPO}

              cd my-app-manifests

              sed -i "s|image:.*|image: ${IMAGE}|g" deployment.yaml

              git config --global --add safe.directory "$PWD"
              git config user.name "jenkins"
              git config user.email "jenkins@local"

              git add deployment.yaml
              git commit -m "Update image to ${IMAGE_TAG}" || echo "No changes to commit"
              git push origin main
            '''
          }
        }
      }
    }
  }
}
