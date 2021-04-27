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
             codeql-runner-linux init --languages java --config-file .github/codeql/codeql-config.yml --codeql-path /opt/codeql/codeql --repository rmahimalur/codeql-runner --github-url https://github.com --github-auth 5e7ab08f53f2becc46284cef4641f4b1441164ad
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
            withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'CBC-DevSecOps', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
             sh"""
             cd codeql-runner
             mkdir -p /tmp/scan-results
             codeql-runner-linux analyze --repository rmahimalur/codeql-runner --github-url https://github.com --commit ca92facbc3eb6b9810172698134efe629eceb5a3  --github-auth 5e7ab08f53f2becc46284cef4641f4b1441164ad --output-dir /tmp/scan-results --ref refs/heads/main
             ls -al /tmp/scan-results
             aws --version
             aws s3 cp /tmp/scan-results/ s3://nimbus-code-scanning-results/ --recursive
             set +e
             cat /tmp/scan-results/java-builtin.sarif | jq -e '.runs[0].results | select(length > 0)'
             if [ "\$?" -eq 0 ]; then exit 1; fi
             """
            }
          }
        }
      }                         
  }
}
