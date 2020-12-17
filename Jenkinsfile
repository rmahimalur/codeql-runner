pipeline {
    agent {
    kubernetes {
        yaml '''
              apiVersion: v1
              kind: Pod
              spec:
                containers:
                  - name: codeql
                    image: "artifactory.cms.gov/nimbus-jenkins-core-docker-local/centos-codeql:latest"
                    tty: true
                    command: ["tail", "-f", "/dev/null"]
                    imagePullPolicy: Always
             '''
             }
         }
    environment {
      JFROG_CLI_HOME_DIR = "${env.WORKSPACE}/.jfrog"
      JFROG_CLI_OFFER_CONFIG=false
      }

  options { timestamps() }

  stages {

    stage('Git Checkout') {
      steps {
          container('codeql'){
             sh"""
             git clone https://github.com/rmahimalur/codeql-runner.git
             """
          }
        }
      }
    stage('Initializing codeql') {
      steps {
          container('codeql'){
             sh"""
             cd codeql-runner
             codeql-runner-linux init --languages java --config-file .github/codeql/codeql-config.yml --codeql-path /opt/codeql/codeql --repository rmahimalur/codeql-runner --github-url https://github.com --github-auth 76c4f99232666938ffe6708fe7022040e06032b8
             """
          }
        }
      }
    stage('monitor and build') {
      steps {
          container('codeql'){
             sh"""
             cd codeql-runner
             chmod +x ${env.WORKSPACE}/codeql-runner/codeql-runner/codeql-env.sh
             . ${env.WORKSPACE}/codeql-runner/codeql-runner/codeql-env.sh
             codeql-runner-linux autobuild --language java
             """
          }
        }
      }
    stage('analyze and upload result to github') {
      steps {
          container('codeql'){
             sh"""
             cd codeql-runner
             mkdir -p /tmp/scan-results
             codeql-runner-linux analyze --repository rmahimalur/codeql-runner --github-url https://github.com --commit 14a56aba645b51488c9c98758c7c2ebecb4177da  --github-auth cf4f2f1c6b56bef90d56d7deb438a08cb4c65b00 --output-dir /tmp/scan-results --ref refs/heads/main
             ls -al /tmp/scan-results
             """
          }
        }
      }
  }
}
