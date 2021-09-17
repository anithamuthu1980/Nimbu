#!/usr/bin/env groovy
// The above line is used to trigger correct syntax highlighting.

pipeline {
  // Lets Jenkins use Docker and Kubernetes for us later.
  agent any

  parameters {
        string(name: 'BUILD_ENV', defaultValue: 'stg', description: 'Set build environment: [dev, test, stg, prod]')
  }

  // triggers {
  //     parameterizedCron(''' H 23 * * * %BUILD_ENV=stg ''')
  // }

  environment {
      def name = "ui-winhub-security"

      PORT_MAPPING = "3050:3333"
      PORT_MAPPING_TEST = "3150:3333"
      PORT_MAPPING_STG = "3550:3333"
      PORT_MAPPING_PROD = "3000:3333"

      IMAGE_NAME = "${name}-img"
      PROCESS_NAME = "${name}"
      SHARED_DIR_CONTAINER = "C:\\Shared"
      SHARED_DIR_HOST = "C:\\Shared\\${params.BUILD_ENV}\\${name}"
      
      //PATH = "C:\\Program Files\\Docker\\Docker\\Resources\\bin;D:\\PATRICK\\installers\\Katalon_Studio_Windows_64-5.9.1;$PATH"
  }

  // If anything fails, the whole Pipeline stops.
  stages {
    stage('Get Common Modules'){
        steps {
            script {
                if (env.BRANCH != null) {
                    echo 'Get Common Modules ('+ BRANCH +')...'
                    powershell 'git submodule foreach -q --recursive git checkout ' + BRANCH
                    powershell 'git submodule foreach -q --recursive git pull '
                } else {
                    echo 'Get Common Modules (master)...'
                    powershell 'git submodule foreach -q --recursive git checkout master'
                    powershell 'git submodule foreach -q --recursive git pull '
                }
            }
        }
    }
    stage('Setup Environment Variables') {
      steps {
        script {
          if (params.BUILD_ENV == 'test') {
            PORT_MAPPING = PORT_MAPPING_TEST
          } else if(params.BUILD_ENV == 'stg') {
            PORT_MAPPING = PORT_MAPPING_STG
          } else if(params.BUILD_ENV == 'prod') {
            PORT_MAPPING = PORT_MAPPING_PROD
          }
        }
      }
    }
    stage('Clean Up Docker Processes'){
      steps {
        script{
          try{
            echo 'Stop Docker Process...'
            bat 'docker stop ' + params.BUILD_ENV + '-' + PROCESS_NAME

            echo 'Remove Docker Process...'
            bat 'docker rm ' + params.BUILD_ENV + '-' + PROCESS_NAME
          }
          catch (Exception e) {
            echo 'Catch docker stop / docker rm error' 
          }
        }
      }
    }
    stage('Build Image') {
      steps {
        echo 'Creating Docker Image...'
        bat 'docker build -f src/CommonModules/deploy/'+ params.BUILD_ENV +'/Dockerfile -t ' + params.BUILD_ENV + '-' + IMAGE_NAME + ' .'
      }
    }
    stage('Run Docker Process') {
      steps {
        echo 'Running Docker Process...'
        bat 'docker run --name ' + params.BUILD_ENV + '-' + PROCESS_NAME + ' -p '+ PORT_MAPPING +' -d ' + params.BUILD_ENV + '-' + IMAGE_NAME
      }
    }
  }
}
