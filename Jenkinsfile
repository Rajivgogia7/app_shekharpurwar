pipeline {
  agent any

  environment {
    scannerHome = 'sonar_scanner_dotnet'
    username = 'shekharpurwar'
    appName = 'Helloworld'
  }

  options {

    timestamps()

    //Set a timeout period for the Pipeline run, after which Jenkins should abort the Pipeline
    timeout(time: 1, unit: 'HOURS')

    buildDiscarder(logRotator(
      // number of build logs to keep
      numToKeepStr: '10',
      // Days to keep histry 
      daysToKeepStr: '30'
    ))
  }

  stages {

    stage("Nuget restore") {
      steps {

        echo "Deployment started for - ${BRANCH_NAME} branch"
        bat "dotnet restore"
      }
    }

    stage('Start sonarqube analysis') {
      when {
        branch "master"
      }

      steps {
        echo "Start sonarqube analysis step"
        withSonarQubeEnv('Test_Sonar') {
          bat "dotnet sonarscanner begin /k:sonar-${userName} /n:sonar-${userName}  -d:sonar.cs.opencover.reportsPaths=TestProject/coverage.opencover.xml -d:sonar.cs.xunit.reportPaths=TestProject/TestResults/testresult.xml/v:1.0"
         
        }
      }
    }

    stage('Code build') {
      steps {
        //Cleans the output of a project
        echo "Clean Previous Build"
        bat "dotnet clean"

        //Builds the project and all of its dependencies
        echo "Code Build"
        bat 'dotnet build -c Release'
      }
    }

    stage('Test Case execution') {
       when {
        branch "master"
      }
      steps {
        //Builds the project and all of its dependencies
        echo "Execute Test Cases"
        bat 'dotnet test /p:CollectCoverage=true /p:CoverletOutputFormat=opencover -l:trx;LogFileName=${appName}.xml'
      }
    }

    stage('Stop sonarqube analysis') {
      when {
        branch "master"
      }

      steps {
        echo "Stop sonarqube analysis"
        withSonarQubeEnv('Test_Sonar') {
          bat "dotnet sonarscanner end"
        }
      }
    }

    stage("Release artifact") {
      when {
        branch "develop"
      }

      steps {
        echo "Release artifact step"
        bat "dotnet publish -c Release -o ${userName}/app/${appName}"
      }
    }

     stage('Kubernetes Deployment') {
        when {
        branch "master"
      }
       steps{
          echo "Start deplying deplyment.yaml file"
          bat "kubectl apply -f deployment.yaml"
          echo "Start deplying serice.yaml file"
          bat "kubectl apply -f service.yaml"
      }
    }
  }

  post {
    always {
      echo 'Workspace Cleanup'
      cleanWs()
    }
  }
}
