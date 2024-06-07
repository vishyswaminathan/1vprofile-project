def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
]




pipeline {
    agent any
    tools {
        maven "MAVEN3"
        jdk "OracleJDK8"

    
    }
    
    
    environment {
        SNAP_REPO = 'vprofile-snapshot'
        NEXUS_USER = 'admin'
        NEXUS_PASS = 'admin123'
        RELEASE_REPO = 'vprofile-release'
        CENTRAL_REPO = 'vpro-maven-central'
        NEXUSIP = '172.31.32.100'
        NEXUSPORT = '8081'
        NEXUS_GRP_REPO = 'vpro-maven-group'
        NEXUS_LOGIN = 'nexuslogin'
        SONARSERVER = 'sonarserver'
        SONARSCANNER = 'sonarscanner'
        NEXUSPASS = credentials('nexuspass')
    }
    
    
    stages {
        stage('Build'){
            steps{
                sh 'mvn -s settings.xml -DskipTests install'
            }

            post {
                success {
                    echo "Now Archiving."
                    archiveArtifacts artifacts: '**/*.war'
                }
            } 
        }

        stage('Test'){
            steps {
                sh 'mvn -s settings.xml test'
            }
         }


        stage('Checkstyle Analysis'){
            steps {
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
        }  
            

        stage('CODE ANALYSIS using SONARQUBE') {
          
		    environment {
                scannerHome = tool "${SONARSCANNER}"
          }

            steps {
               withSonarQubeEnv("${SONARSERVER}") {
                   sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile-repo \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
            }

            
            }
          }

        stage("Quality Gate") {
            steps{
                timeout(time: 1, unit: 'MINUTES') {



                 waitForQualityGate abortPipeline: true
              


            }
        } 
          

        }

        stage("Upload artifact") {
            steps{
                nexusArtifactUploader(
                nexusVersion: 'nexus3',
                protocol: 'http',
                nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
                groupId: 'QA',
                version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                repository: "${RELEASE_REPO}",
                credentialsId: "${NEXUS_LOGIN}",
                artifacts: [
                    [artifactId: 'vproapp',
                     classifier: '',
                     file: 'target/vprofile-v2.war',
                     type: 'war']
                ]     
               )  
            }
        }
        


        stage('Ansible Deploy to staging'){
            steps {
                ansiblePlaybook([
                inventory   : 'ansible/stage.inventory',
                playbook    : 'ansible/site.yml',
                installation: 'ansible',
                colorized   : true,
			    credentialsId: 'applogin',
			    disableHostKeyChecking: true,
                extraVars   : [
                   	USER: "admin",
                    PASS: "${NEXUSPASS}",
			        nexusip: "172.31.32.100",
			        reponame: "vprofile-release",
			        groupid: "QA",
			        time: "${env.BUILD_TIMESTAMP}",
			        build: "${env.BUILD_ID}",
                    artifactid: "vproapp",
			        vprofile_version: "vproapp-${env.BUILD_ID}-${env.BUILD_TIMESTAMP}.war"
                ]
             ])
            }
        }


           



        }


    }
    post {
        always {
            echo 'Slack Notifications.'
            slackSend channel: '#jenkinscicd',
                color: COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}" 
        }
    }





}