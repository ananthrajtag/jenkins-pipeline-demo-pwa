#!groovy

pipeline {

    agent {
      /*
      docker {
        image 'jenkinsslave:latest'
        registryUrl 'https://hub.docker.com/'
        registryCredentialsId 'sivisoft:Pass123!@#'
        args '-v /home/centos/.ivy2:/home/jenkins/.ivy2:rw -v jenkins_opt:/usr/local/bin/opt -v jenkins_apijenkins:/home/jenkins/config -v jenkins_logs:/var/logs -v jenkins_awsconfig:/home/jenkins/.aws --privileged=true -u jenkins:jenkins'
      } */
      label 'LINUX' 
        //image 'jenkinsslave:latest'
       // label 'LINUX'
       // registryUrl 'https://hub.docker.com/'
       // registryCredentialsId 'sivisoft:Pass123!@#'
        //args '-v /home/centos/.ivy2:/home/jenkins/.ivy2:rw -v jenkins_opt:/usr/local/bin/opt -v jenkins_apijenkins:/home/jenkins/config -v jenkins_logs:/var/logs -v jenkins_awsconfig:/home/jenkins/.aws --privileged=true -u jenkins:jenkins'
    }
  
    environment {
        APP_NAME = 'jenkins-pipeline-demo-pwa'
        BUILD_NUMBER = "${env.BUILD_NUMBER}"
        IMAGE_VERSION="v_${BUILD_NUMBER}"
        //GIT_URL="git@github.yourdomain.com:mpatel/${APP_NAME}.git"
        GIT_URL="https://github.com/ananthrajtag/${APP_NAME}.git"
        GIT_CRED_ID='github_ananthraj'
        REPOURL = 'sivisoft'
        SBT_OPTS='-Xmx1024m -Xms512m'
        JAVA_OPTS='-Xmx1024m -Xms512m'
        WS_PRODUCT_TOKEN='FJbep9fKLeJa/Cwh7IJbL0lPfdYg7q4zxvALAxWPLnc='
        WS_PROJECT_TOKEN='zwzxtyeBntxX4ixHD1iE2dOr4DVFHPp7D0Czn84DEF4='
        HIPCHAT_TOKEN = 'SpVaURsSTcWaHKulZ6L4L+sjKxhGXCkjSbcqzL42ziU='
        HIPCHAT_ROOM = 'NotificationRoomName'
    }

    options {
        buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '10', numToKeepStr: '20'))
        timestamps()
        //retry(3)
        timeout time:10, unit:'MINUTES'
    }
    parameters {
        string(defaultValue: "develop", description: 'Branch Specifier', name: 'SPECIFIER')
        booleanParam(defaultValue: false, description: 'Deploy to QA Environment ?', name: 'DEPLOY_QA')
        booleanParam(defaultValue: false, description: 'Deploy to UAT Environment ?', name: 'DEPLOY_UAT')
        booleanParam(defaultValue: false, description: 'Deploy to PROD Environment ?', name: 'DEPLOY_PROD')
    }

    stages {
        stage("Initialize") {
            steps {
                script {
                    //notifyBuild('STARTED')
                    echo "${BUILD_NUMBER} - ${env.BUILD_ID} on ${env.JENKINS_URL}"
                    echo "Branch Specifier :: ${params.SPECIFIER}"
                    echo "Deploy to QA? :: ${params.DEPLOY_QA}"
                    echo "Deploy to UAT? :: ${params.DEPLOY_UAT}"
                    echo "Deploy to PROD? :: ${params.DEPLOY_PROD}"
                    sh 'rm -rf target/universal/*.zip'
                }
            }
        }
        stage('Checkout') {
            steps {
                echo 'Checkout Repo'
                git branch: "${params.SPECIFIER}", url: "${GIT_URL}"
            }
        }
        stage('Build') {
            steps {
              echo 'npm build'
               // sh 'npm install'
                //sh 'ng lint'
                //sh 'ng build --prod'
            }
        }
      
      /*
        stage('Static Code Coverage Analysis') {
            parallel {
              stage('Execute Whitesource Analysis') {
                  steps {
                      whitesource jobApiToken: '', jobCheckPolicies: 'global', jobForceUpdate: 'global', libExcludes: '', libIncludes: '', product: "$WS_PRODUCT_TOKEN", productVersion: '', projectToken: "$WS_PROJECT_TOKEN", requesterEmail: ''
                  }
              }
              stage('SonarQube analysis') {
                  steps {
                     script {
                     // sh "/usr/bin/sonar-scanner -X"
                    def scannerHome = tool('SonarQube')
                    withSonarQubeEnv {
			            sh("""
                    ${scannerHome}/bin/sonar-scanner -X
                    """)
                    }
                     }
                  }
              }
            }
        }
      
      */
      //Publish report
      
      stage('Publish Reports') {
    parallel {
        stage('Publish FindBugs Report') {
            steps {
                step([$class: 'FindBugsPublisher', canComputeNew: false, defaultEncoding: '', excludePattern: '', healthy: '', includePattern: '', pattern: 'target/scala-2.11/findbugs/report.xml', unHealthy: ''])
            }
        }
        stage('Publish Junit Report') {
            steps {
                junit allowEmptyResults: true, testResults: 'target/test-reports/*.xml'
            }
        }
        stage('Publish Junit HTML Report') {
            steps {
                publishHTML target: [
                        allowMissing: true,
                        alwaysLinkToLastBuild: false,
                        keepAll: true,
                        reportDir: 'target/reports/html',
                        reportFiles: 'index.html',
                        reportName: 'Test Suite HTML Report'
                ]
            }
        }
        stage('Publish Coverage HTML Report') {
            steps {
                publishHTML target: [
                        allowMissing: true,
                        alwaysLinkToLastBuild: false,
                        keepAll: true,
                        reportDir: 'target/scala-2.11/scoverage-report',
                        reportFiles: 'index.html',
                        reportName: 'Code Coverage'
                ]
            }
        }
        stage('Execute Whitesource Analysis') {
            steps {
                whitesource jobApiToken: '', jobCheckPolicies: 'global', jobForceUpdate: 'global', libExcludes: '', libIncludes: '', product: "${env.WS_PRODUCT_TOKEN}", productVersion: '', projectToken: "${env.WS_PROJECT_TOKEN}", requesterEmail: ''
            }
        }
        stage('SonarQube analysis') {
                  steps {
                     script {
                     // sh "/usr/bin/sonar-scanner -X"
                    def scannerHome = tool('SonarQube')
                    withSonarQubeEnv {
			            sh("""
                    ${scannerHome}/bin/sonar-scanner -X
                    """)
                    }
                     }
                  }
              }
        stage('ArchiveArtifact') {
            steps {
              echo 'Archive Artifact completed'
                //archiveArtifacts '**/target/universal/*.zip'
            }
        }
    }
}
      
      ///
      /*
         stage('Docker Tag & Push') {
             steps {
                 script {
                     branchName = getCurrentBranch()
                     shortCommitHash = getShortCommitHash()
                     IMAGE_VERSION = "${BUILD_NUMBER}-" + branchName + "-" + shortCommitHash
                     sh 'eval $(aws ecr get-login --no-include-email --region us-east-1)'
                     sh "docker-compose build"
                     sh "docker tag ${REPOURL}/${APP_NAME}:latest ${REPOURL}/${APP_NAME}:${IMAGE_VERSION}"
                     sh "docker push ${REPOURL}/${APP_NAME}:${IMAGE_VERSION}"
                     sh "docker push ${REPOURL}/${APP_NAME}:latest"

                     sh "docker rmi ${REPOURL}/${APP_NAME}:${IMAGE_VERSION} ${REPOURL}/${APP_NAME}:latest"
                 }
             }
         }
          */
        stage('Deploy - CI') {
            steps {
                echo "Deploying to CI Environment."
            }
        }

        stage('Deploy - QA') {
            when {
                expression {
                    params.DEPLOY_QA == true
                }
            }
            steps {
                echo "Deploy to QA..."
            }
        }
        stage('Deploy - UAT') {
            when {
                expression {
                    params.DEPLOY_UAT == true
                }
            }
            steps {
                echo "Deploy to UAT..."
            }
        }
        stage('Deploy - Production') {
            when {
                expression {
                    params.DEPLOY_PROD == true
                }
            }
            steps {
                echo "Deploy to PROD..."
            }
        }
    }

    post {
        /*
         * These steps will run at the end of the pipeline based on the condition.
         * Post conditions run in order regardless of their place in pipeline
         * 1. always - always run
         * 2. changed - run if something changed from last run
         * 3. aborted, success, unstable or failure - depending on status
         */
        always {
            echo "I AM ALWAYS first"
           echo 'One way or another, I have finished' 
            deleteDir() /* clean up our workspace */ 
            //notifyBuild("${currentBuild.currentResult}")
        }
        aborted {
            echo "BUILD ABORTED"
        }
        success {
            echo "BUILD SUCCESS"
            echo "Keep Current Build If branch is master"
//            keepThisBuild()
        }
        unstable {
            echo "BUILD UNSTABLE"
        }
        failure {
            echo "BUILD FAILURE"
        }
    }
}

def keepThisBuild() {
    currentBuild.setKeepLog(true)
    currentBuild.setDescription("Test Description")
}

def getShortCommitHash() {
    return sh(returnStdout: true, script: "git log -n 1 --pretty=format:'%h'").trim()
}

def getChangeAuthorName() {
    return sh(returnStdout: true, script: "git show -s --pretty=%an").trim()
}

def getChangeAuthorEmail() {
    return sh(returnStdout: true, script: "git show -s --pretty=%ae").trim()
}

def getChangeSet() {
    return sh(returnStdout: true, script: 'git diff-tree --no-commit-id --name-status -r HEAD').trim()
}

def getChangeLog() {
    return sh(returnStdout: true, script: "git log --date=short --pretty=format:'%ad %aN <%ae> %n%n%x09* %s%d%n%b'").trim()
}

def getCurrentBranch () {
    return sh (
            script: 'git rev-parse --abbrev-ref HEAD',
            returnStdout: true
    ).trim()
}

def isPRMergeBuild() {
    return (env.BRANCH_NAME ==~ /^PR-\d+$/)
}

def notifyBuild(String buildStatus = 'STARTED') {
    // build status of null means successful
    buildStatus = buildStatus ?: 'SUCCESS'

    def branchName = getCurrentBranch()
    def shortCommitHash = getShortCommitHash()
    def changeAuthorName = getChangeAuthorName()
    def changeAuthorEmail = getChangeAuthorEmail()
    def changeSet = getChangeSet()
    def changeLog = getChangeLog()

    // Default values
    def colorName = 'RED'
    def colorCode = '#FF0000'
    def subject = "${buildStatus}: '${env.JOB_NAME} [${env.BUILD_NUMBER}]'" + branchName + ", " + shortCommitHash
    def summary = "Started: Name:: ${env.JOB_NAME} \n " +
            "Build Number: ${env.BUILD_NUMBER} \n " +
            "Build URL: ${env.BUILD_URL} \n " +
            "Short Commit Hash: " + shortCommitHash + " \n " +
            "Branch Name: " + branchName + " \n " +
            "Change Author: " + changeAuthorName + " \n " +
            "Change Author Email: " + changeAuthorEmail + " \n " +
            "Change Set: " + changeSet

    if (buildStatus == 'STARTED') {
        color = 'YELLOW'
        colorCode = '#FFFF00'
    } else if (buildStatus == 'SUCCESS') {
        color = 'GREEN'
        colorCode = '#00FF00'
    } else {
        color = 'RED'
        colorCode = '#FF0000'
    }

    // Send notifications
    hipchatSend(color: color, notify: true, message: summary, token: "${env.HIPCHAT_TOKEN}",
        failOnError: true, room: "${env.HIPCHAT_ROOM}", sendAs: 'Jenkins', textFormat: true)

    if (buildStatus == 'FAILURE') {
        emailext attachLog: true, body: summary, compressLog: true, recipientProviders: [brokenTestsSuspects(), brokenBuildSuspects(), culprits()], replyTo: 'noreply@yourdomain.com', subject: subject, to: 'mpatel@yourdomain.com'
    }
}

