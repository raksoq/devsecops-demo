pipeline {
  environment {
    // nodeport address of argocd service
    ARGO_SERVER = '35.202.161.131:32100'
  }
  agent {
    kubernetes {
      yamlFile 'build-agent.yaml'
      defaultContainer 'maven'
      idleMinutes 1
    }
  }
  stages {
    stage('Build') {
      parallel {
        stage('Compile') {
          steps {
            container('maven') {
              sh 'mvn compile'
            }
          }
        }
      }
    }
    stage('Static Analysis') {
      parallel {
        stage('Unit Tests') {
          steps {
            container('maven') {
              sh 'mvn test'
            }
          }
        }
        stage('OSS License Checker failed') {
          steps {
            container('licensefinder') {
              sh 'ls -al'
              sh "echo 'stopped working after changes in section 6 done'"
              // sh '''#!/bin/bash --login
              //       /bin/bash --login
              //       rvm use default
              //       gem install license_finder
              //       license_finder
              // '''
            }
          }
        }
        stage('SCA') {
          steps {
            container('maven') {
              catchError(buildResult: 'SUCCESS', stageResult:'FAILURE') {
                sh 'mvn org.owasp:dependency-check-maven:check'
              }
            }
          }
          post {
            always {
              archiveArtifacts allowEmptyArchive: true,
              artifacts: 'target/dependency-check-report.html', fingerprint:
              true, onlyIfSuccessful: true
              // dependencyCheckPublisher pattern: 'report.xml'
            }
          }
       }
      }
    }
    stage('SAST failed') {
        steps {
          container('slscan') {
            sh "echo 'stopped working after changes in section 6 done'"
            // sh 'scan --type java,depscan --build'
          }
        }
        post {
          success {
            archiveArtifacts allowEmptyArchive: true,
            artifacts: 'reports/*', fingerprint: true, onlyIfSuccessful:
            true
          }
        }
    }
    stage('Package') {
      parallel {
        stage('Create Jarfile') {
          steps {
            container('maven') {
              sh 'mvn package -DskipTests'
            }
          }
        }
        stage('Docker BnP') {
         steps {
          container('kaniko') {
            script {
              sh '''
              /kaniko/executor --dockerfile `pwd`/Dockerfile \
                --context `pwd` \
                --destination=oskarq/devsecops-demo:latest
              '''
            }
          }           
         }
        }
      }
    }
    stage('Image Analysis') {
      parallel {
        stage('Image Linting') {
          steps {
            container('docker-tools') {
            sh 'dockle docker.io/oskarq/devsecops-demo:latest'
            }
          }
        }
        stage('Image Scan failed') {
          steps {
            container('docker-tools') {
            // sh 'trivy image --exit-code 1 oskarq/devsecops-demo:latest'
            sh "echo 'stopped working after changes in section 6 done'"
            }
          }
        }
      }
    }

    stage('Deploy to Dev') {
      environment {
        AUTH_TOKEN = credentials('argocd-jenkins-deployer-token')
      }
      steps {
        container('docker-tools') {
          sh 'docker run -t oskarq/argocd-cli:2.10.1 argocd app sync devsecops-demo --insecure --server $ARGO_SERVER --auth-token $AUTH_TOKEN'
          sh 'docker run -t oskarq/argocd-cli:2.10.1 argocd app wait devsecops-demo --health --timeout 300 --insecure --server $ARGO_SERVER --auth-token $AUTH_TOKEN'
        }
      }
    }
  }
}
