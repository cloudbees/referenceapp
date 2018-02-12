
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
          sh 'mvn clean deploy'
          sh "mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.4.0.905:sonar -Dsonar.host.url=http://192.168.64.7:32738/ -Dsonar.login=3d8b59b66720958723ed45692b2616d9de014289"
          sh "docker build -f Dockerfile.release -t $JENKINS_X_DOCKER_REGISTRY_SERVICE_HOST:$JENKINS_X_DOCKER_REGISTRY_SERVICE_PORT/$ORG/$APP_NAME:\$(cat VERSION) ."
          sh "docker push $JENKINS_X_DOCKER_REGISTRY_SERVICE_HOST:$JENKINS_X_DOCKER_REGISTRY_SERVICE_PORT/$ORG/$APP_NAME:\$(cat VERSION)"
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
