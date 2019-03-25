pipeline {
  agent none
  stages {
    stage('Build') {
      agent { label 'dotnetcore21' }
      steps {
        sh "dotnet build"
      }
    }
    stage('Build Image') {
      agent { label 'base' }
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject("peopleapi") {
              openshift.selector("bc", "peopleapi-bc").startBuild("--wait=true")
            }
          }
        }
      }
    }
    stage('Scan Codebase') {
      agent { label 'sonar-dotnet' }
      steps {
        withSonarQubeEnv('SonarQube-Server') {
          sh "dotnet sonarscanner begin /k:peopleapi /d:sonar.host.url=$SONAR_HOST_URL /d:sonar.login=$SONAR_AUTH_TOKEN"
          sh "dotnet build"
          sh "dotnet sonarscanner end /d:sonar.login=$SONAR_AUTH_TOKEN"
        }
      }
    }
    /*  You may need a docker-in-docker to run this on a Jenkins slave agent and do not like doing that */
    /*
    stage('Microscanner security scan'){
      steps {
        aquaMicroscanner imageName: 'peopleapi/peopleapi:latest', notCompliesCmd: 'exit 1', onDisallowed: 'fail', outputFormat: 'html'
      }
    }
    */
  }
}
