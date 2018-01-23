
pipeline {
  agent {
    label "jenkins-maven"
  }

  environment {
    ORG 		= 'jenkinsx'
    APP_NAME    = 'my-spring-boot-demo2'
  }

  stages {

    stage('Build Release') {
      steps {
        container('maven') {
          // so we can retrieve the version in later steps
          sh "echo \$(jx-release-version) > VERSION"
          sh "mvn versions:set -DnewVersion=\$(jx-release-version)"
        }

        dir ('./charts/my-spring-boot-demo2') {
          container('maven') {
            // until kubernetes plugin supports init containers https://github.com/jenkinsci/kubernetes-plugin/pull/229/
            sh 'cp /root/netrc/.netrc ~/.netrc'

            sh "make tag"
          }
        }

        container('maven') {
          sh 'mvn clean deploy'
          sh "docker build -f Dockerfile.jenkins -t $JENKINS_X_DOCKER_REGISTRY_SERVICE_HOST:$JENKINS_X_DOCKER_REGISTRY_SERVICE_PORT/$ORG/$APP_NAME:\$(cat VERSION) ."
          sh "docker push $JENKINS_X_DOCKER_REGISTRY_SERVICE_HOST:$JENKINS_X_DOCKER_REGISTRY_SERVICE_PORT/$ORG/$APP_NAME:\$(cat VERSION)"
        }
      }
    }
    stage('Deploy Staging') {

      steps {
        dir ('./charts/my-spring-boot-demo2') {
          container('maven') {

            sh 'make release'
            sh 'helm install . --namespace staging --name example-release'
            sh 'exposecontroller --namespace staging --http' // until we switch to git environments where helm hooks will expose services
          }
        }
      }
    }
  }
}
