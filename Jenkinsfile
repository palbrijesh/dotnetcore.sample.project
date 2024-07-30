pipeline {
    
    agent any
    
    environment{
        SONAR_HOME= tool "sonar"
        EMAIL_TO = 'brijesh.pal@hotmail.com'
    }
    stages {
        stage('Clone code from GitHub') {
            steps {
                git url: "https://github.com/palbrijesh/dotnetcore.sample.project.git", branch: "master"
            }
        }
        stage('SonarQube Quality Analysis') {
            steps {
                withSonarQubeEnv("sonar"){
                    sh "$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=dotnetcore.sample.project -Dsonar.projectKey=dotnetcore.sample.project"
                }
            }
        }
        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'odc'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                echo "Dependency Check --- Done"
            }
        }
        stage('Sonar Quality Gate Scan') {
            steps {
                timeout(time: 2, unit: "MINUTES"){
                    waitForQualityGate abortPipeline: false
                }
            }
        }
        stage('Trivy File System Scan') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
        stage('Deploy') {
            steps {
                echo "Code deployment"
            }
        }
    }
    
    post {
        success {
            notifySuccessful()
        }
        failure {
            notifyFailed()
        }
        unstable {
            notifyUnstable()
        }
        changed {
            notifyChanged()
        }
    }
}

def notifySuccessful() { 
    emailext (
      subject: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
      body: """<p>SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
      to: "$EMAIL_TO"
    )
 }

def notifyFailed() {
   emailext (
      subject: "【Jenkins CI】FAILED: Project '${env.JOB_NAME}'",
      body: """<h3>Build Failed:  '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</h3>
        <p><b>Project Name :</b> '${env.JOB_NAME}'</p>
        <p><b>Build Times :</b> '${env.BUILD_NUMBER}'</p>
        <!--<p><b>The author :</b> '${env.CHANGE_AUTHOR}'</p>
        <p><a href='${env.BUILD_URL}'>Click for details</a></p>-->
        <p><b>Cause :</b> '${CAUSE}'</p>
        <p><b>Console Message :</b> Please check the attachment</p>""",
       to: "$EMAIL_TO",
       attachLog: true, 
       compressLog: true
    )
}

def notifyUnstable() {
    emailext body: 'Check console output at $BUILD_URL to view the results. \n\n ${CHANGES} \n\n -------------------------------------------------- \n${BUILD_LOG, maxLines=100, escapeHtml=false}', 
                    to: "${EMAIL_TO}", 
                    subject: 'Unstable build in Jenkins: $PROJECT_NAME - #$BUILD_NUMBER'
}

def notifyChanged() {
    emailext body: 'Check console output at $BUILD_URL to view the results.', 
                    to: "${EMAIL_TO}", 
                    subject: 'Jenkins build is back to normal: $PROJECT_NAME - #$BUILD_NUMBER'
}
