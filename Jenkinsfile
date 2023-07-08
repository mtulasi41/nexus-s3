pipeline {
    agent any
    environment {
        nexusUrl = '13.127.248.17:8081'
        nexusRepository = 'maven-snapshots'
        groupId = 'com.mkyong'
        artifactId = 'counterwebapp'
        version = '1.0-SNAPSHOT'
        classifier = ''
        NEXUS_USERNAME = 'admin'
        NEXUS_PASSWORD = 'nexus123'
        S3_BUCKET = "nexusartifacts-s3"
        AWS_ACCESS_KEY_ID = 'AKIAXAGHLNQS46WROVKK'
        AWS_SECRET_ACCESS_KEY = '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'
        }
    
def getLatestArtifactVersion() {
    def response = sh(script: "curl -s -u $NEXUS_USERNAME:$NEXUS_PASSWORD ${NEXUS_URL}/${nexusRepository}/${groupId}/${artifactId}/${version}/${classifier}", returnStdout: true)
    def latestVersion = response.returnStdout =~ "<latest>(.*?)</latest>"
    return latestVersion[0][1]
}
    def downloadArtifact(version) {
    sh "curl -u $NEXUS_USERNAME:$-O ${NEXUS_URL}/${nexusRepository}/${groupId}/${artifactId}/${version}/${artifactId}-$version.war"
}
    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven "Maven3.9.3"
    }

    stages {
        stage('checkout'){
            steps {
                checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/mtulasi41/nexus-s3.git']])
            }
        }
        stage('Build') {
            steps {
                
               // Run Maven on a Unix agent.
                sh "mvn clean package"

            }
        }
       stage('nexusupload'){
            steps{
                nexusArtifactUploader artifacts: [[artifactId: 'counterwebapp', classifier: '', file: '/var/lib/jenkins/workspace/pipeline-nexus-s3/target/CounterWebApp.war', type: 'WAR']], credentialsId: 'nexus', groupId: 'com.mkyong', nexusUrl: '13.127.248.17:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'maven-snapshots', version: '1.0-SNAPSHOT'
            }          
        }
        stage('Fetch latest artifact version') {
            steps {
                script {
                    def latestVersion = getLatestArtifactVersion()
                    echo "Latest artifact version: $latestVersion"
                    downloadArtifact(latestVersion)
                }
            }
        }
   }
}
