
def checkSonar() {
    withSonarQubeEnv('default') {
      sh 'mvn sonar:sonar'
      timeout(time: 5, unit: 'MINUTES') {
        def qualityGate = waitForQualityGate()
        if (qualityGate.status != 'OK') {
          error "Pipeline aborted due to quality gate failure: ${qualityGate.status}"
        }
      }
    }
}

def buildAndPublishContainer() {
  sh "docker build -f Dockerfile.release -t $JENKINS_X_DOCKER_REGISTRY_SERVICE_HOST:$JENKINS_X_DOCKER_REGISTRY_SERVICE_PORT/$ORG/$APP_NAME:\$(cat VERSION) ."
  sh "docker push $JENKINS_X_DOCKER_REGISTRY_SERVICE_HOST:$JENKINS_X_DOCKER_REGISTRY_SERVICE_PORT/$ORG/$APP_NAME:\$(cat VERSION)" 
}

pipeline {
  agent {
    label "jenkins-maven"
  }

  environment {
    ORG 		= 'jenkinsx'
    APP_NAME    = 'referenceapp'
  }

  stages {

    stage('Build Release') {
      steps {
        container('maven') {
          // ensure we're not on a detached head
          sh "git checkout master"

          // until we switch to the new kubernetes / jenkins credential implementation use git credentials store
          sh "git config credential.helper store"

          // so we can retrieve the version in later steps
          sh "echo \$(jx-release-version) > VERSION"
          sh "mvn versions:set -DnewVersion=\$(jx-release-version)"
        }

        //dir ('./charts/referenceapp') {
        //  container('maven') {
        //    sh "make tag"
        //  }
        //}

        container('maven') {
          sh 'mvn clean test'
          checkSonar()
          sh 'mvn deploy'
          buildAndPublishContainer()
        }
      }
    }
    stage('Deploy Staging') {

      steps {
        dir ('./charts/referenceapp') {
          container('maven') {

            sh 'make release'
            sh 'jx promote --all-auto --version \$(cat ../../VERSION)'
          }
        }
      }
    }
  }
}
