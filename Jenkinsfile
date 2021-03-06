pipeline {
  agent any
  stages {
    stage('Container Build') {
      parallel {
        stage('Container Build') {
          steps {
            echo 'Building...'
          }
        }
        stage('Building tng-vnv-dsm...') {
          steps {
            sh 'docker build --no-cache -t registry.sonata-nfv.eu:5000/tng-vnv-dsm .'
          }
        }
      }
    }
    stage('Unit Test') {
      parallel {
        stage('Unit Tests') {
          steps {
            echo 'Performing Unit Tests'
          }
        }
        stage('Running Unit Tests') {
          steps {
			echo 'Performing Unit Tests'
           }
        }
      }
    }
    stage('Containers Publication') {
      parallel {
        stage('Containers Publication') {
          steps {
            echo 'Publication of containers in local registry....'
          }
        }
        stage('Publishing tng-vnv-dsm') {
          steps {
            sh 'docker push registry.sonata-nfv.eu:5000/tng-vnv-dsm'
          }
        }
      }
    }


  stage('Deployment in Pre-Integration SP and VnV Environment') {
    parallel {
      stage('Deployment in Pre-Integration SP and VnV Environment') {
        steps {
          echo 'Deploying in pre-integration...'
        }
      }
      stage('Deploying') {
        steps {
		  sh 'docker push registry.sonata-nfv.eu:5000/tng-vnv-dsm:latest'
          sh 'rm -rf tng-devops || true'
          sh 'git clone https://github.com/sonata-nfv/tng-devops.git'
          dir(path: 'tng-devops') {
           sh 'ansible-playbook roles/sp.yml -i environments -e "target=pre-int-sp component=tng-vnv-dsm"'
           sh 'ansible-playbook roles/vnv.yml -i environments -e "target=pre-int-vnv component=tng-vnv-dsm"'
          }
        }
      }
    }
  }
  
  stage('Promoting containers to integration env') {
    when {
       branch 'master'
    }
    parallel {
        stage('Publishing containers to int') {
            steps {
            echo 'Promoting containers to integration'
            }
         }
        stage('tng-vnv-dsm') {
		    steps {
				sh 'docker tag registry.sonata-nfv.eu:5000/tng-vnv-dsm:latest registry.sonata-nfv.eu:5000/tng-vnv-dsm:int'
				sh 'docker push registry.sonata-nfv.eu:5000/tng-vnv-dsm:int'
				sh 'rm -rf tng-devops || true'
				sh 'git clone https://github.com/sonata-nfv/tng-devops.git'
				dir(path: 'tng-devops') {
					sh 'ansible-playbook roles/sp.yml -i environments -e "target=int-sp component=tng-vnv-dsm"'
					sh 'ansible-playbook roles/vnv.yml -i environments -e "target=int-vnv component=tng-vnv-dsm"'
				}
			}
        }
    }
  }
  }
  post {
    always {
      junit(allowEmptyResults: true, testResults: 'spec/reports/*.xml')
    }
     success {
        emailext(from: "jenkins@sonata-nfv.eu",
        to: "pstav@unipi.gr",
        subject: "SUCCESS: ${env.JOB_NAME}/${env.BUILD_ID} (${env.BRANCH_NAME})",
        body: "${env.JOB_URL}")
     }
     failure {
        emailext(from: "jenkins@sonata-nfv.eu",
        to: "pstav@unipi.gr",
        subject: "FAILURE: ${env.JOB_NAME}/${env.BUILD_ID} (${env.BRANCH_NAME})",
        body: "${env.JOB_URL}")
        }
  }
}
