pipeline {
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
        stage('OSS License Checker') {
          steps {
            container('licensefinder') {
              sh '''
              # Set the PATH correctly for RVM
              export PATH="/usr/share/rvm/gems/ruby-3.1.1/bin:$PATH"
              # Ensure using the correct Ruby version
              rvm use ruby-3.1.1
              # Install and run license_finder
              gem install license_finder
              license_finder
              '''
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
    stage('SAST') {
        steps {
          container('slscan') {
            sh 'scan --type java,depscan --build'
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

    stage('Deploy to Dev') {
      steps {
        // TODO
        sh "echo done"
      }
    }
  }
}
