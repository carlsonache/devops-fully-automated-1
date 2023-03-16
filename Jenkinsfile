def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
]

pipeline {
    agent any

    environment {
        WORKSPACE = "${env.WORKSPACE}"
    }

    tools {
        maven 'localMaven'
        jdk 'localJdk'
    }

    stages {
        stage('Git checkout') {
            steps {
                echo 'Cloning the application code...'
                git branch: 'main', url: 'https://github.com/carlsonache/devops-fully-automated-1.git'

            }
        }

        stage('Build') {
            steps {
                sh 'java -version'
                sh 'mvn -U clean package'
            }

            post {
                success {
                    echo 'archiving....'
                    archiveArtifacts artifacts: '**/*.war', followSymlinks: false
                }
            }
        }

        stage('Unit Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Integration Test') {
            steps {
                sh 'mvn verify -DskipUnitTests'
            }
        }
        stage('Checkstyle Code Analysis') {
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }

        stage('SonarQube Scan') {
          steps {
            sh """mvn sonar:sonar \
                      -Dsonar.projectKey=sonar \
                      -Dsonar.host.url=http://54.211.144.113:9000 \
                      -Dsonar.login=db0767264ee7725fd4f468b10509dfc2a1d04b47"""
          }
        }

        stage('Upload to Artifactory to Nexus') {
          steps {
            sh "mvn clean deploy -DskipTests -s settings.xml -X"
          }

        }

    stage('Deploy to DEV') {
      environment {
        HOSTS = "dev"
      }
      steps {
        sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
      }

    }
    stage('Approval for dev') {
      steps {
        input('Do you want to proceed?')
      }
    }
    stage('Deploy to Stage') {
      environment {
        HOSTS = "dev"
      }
      steps {
        sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
      }

    }
    stage('Approval for Stage') {
      steps {
        input('Do you want to proceed?')
      }
    }
    stage('Deploy to PROD') {
      environment {
        HOSTS = "prod"
      }
      steps {
        sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
      }
    }
  }
}

