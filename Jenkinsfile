@Library('Shared') _
pipeline {
   agent any 
 
   environment{
      SONAR_HOME = tool "Sonar"
  }

  parameters {
       string(name: 'FRONTEND_DOCKER_TAG', defaultvalue: '', description: 'Setting docker image for latest push')
       string(name: 'BACKEND_DOCKER_TAG', defaultvalue: '', description: 'Setting docker image for latest push')
  }
  
   stages {
     stage("validate parameters") {
        steps {
          script {
            if (params.FRONTEND_DOCKER_TAG == '' || params.BACKEND_DOCKER_TAG == '') {
            error("FRONTEND_DOCKER_TAG and BACKEND_DOCKER_TAG must be provided.")
          }
       } 
     }
    }

     stage("Workspace Cleanup") {
        steps {
          script {
            cleanWs()
        }
      }
    }
     
     stage("Git: code checkout") {
       steps {
         script {
              code_checkout("https://github.com/Chhaya178/travel-devops.git", "main")
          }
       }
     }

     stage("Trivy: Filesystem scan") {
        steps {
           script {
              trivy_scan()
            }
         }
      }

     stage("OWASP: Dependency check") {
        steps {
           script {
               owasp_dependency()
          }
        } 
     }

     stage("SonarQube: Code Analysis") {
       steps {
          script {
             code_analysis("Sonar", "wanderlust", "wanderlust")
           }
         }
      }

     stage("SonarQube: Code Quality Gates") {
        steps {
           script {
              sonarqube_code_quality()
           }
        }
      }

     stage("Exporting environment variables") {
        parallel {
          stage("backend env setup") {
             steps {
               script {
                  dir("Automations") {
                     sh "bash updatebackendnew.sh"
                 }
               }
            }
         }
          stage("frontend env setup") {
            steps {
               script {
                  dir("Automations") {
                      sh "bash updatefrontendnew.sh"
               } 
             }
           }
         }
       }
     }

     stage("Docker: Build Images") {
        steps {
           script {
             dir('backend') {
               docker_build("travel-backend", "${params.BACKEND_DOCKER_TAG}", "shadowigw")
           }
             
            dir('frontend') {
                docker_build("travel-frontend", "${params.FRONTEND_DOCKER_TAG}", "shadowigw")
            }
         }
       }
     }


    stage("Docker: Push to DockerHub") {
       steps {
           script {
              docker_push("travel-backend", "${params.BACKEND_DOCKER_TAG}", "shadowigw")
              docker_push("travel-frontend", "${params.FRONTEND_DOCKER_TAG}", "shadowigw")
            }
         }
      }

    }

   post{
     success{
       archiveArtifacts artifacts: '*.xml', followSymlinks: false
       build job: "Travel-CD", parameters: [
          string(name: 'FRONTEND_DOCKER_TAG', value: "${params.FRONTEND_DOCKER_TAG}")
          string(name: 'BACKEND_DOCKER_TAG', value: "${params.BACKEND_DOCKER_TAG}")
      ]
    }
  }
}
