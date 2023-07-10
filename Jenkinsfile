pipeline {
    agent any
    environment {
        NEXUS_URL = "65.2.135.243:8081"  
        NEXUS_USERNAME = 'admin'  
        NEXUS_PASSWORD = 'nexus123'  
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
                nexusArtifactUploader artifacts: [[artifactId: 'counterwebapp', classifier: '', file: '/var/lib/jenkins/workspace/pipeline-nexus-s3/target/CounterWebApp.war', type: 'war']], credentialsId: 'nexus', groupId: 'com.mkyong', nexusUrl: '65.2.135.243:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'maven-snapshots', version: '1.0-SNAPSHOT'
            }          
        }
        stage('Pull Latest Artifacts') {
            steps {
                script {
                    def nexusAuth = "${NEXUS_USERNAME}:${NEXUS_PASSWORD}".bytes.encodeBase64().toString()
                    def groupId = 'com.mkyong'  // Group ID of the artifacts to be pulled
                    def artifactId = 'counterwebapp'  // Artifact ID of the artifacts to be pulled
                    def repository = 'maven-snapshots'  // Nexus repository name
                    
                    // Construct the Nexus search URL to retrieve the latest version of the artifact
                    def searchUrl = "${NEXUS_URL}/service/rest/v1/search/assets?repository=${repository}&group=${groupId}&name=${artifactId}&sort=version&direction=desc&limit=1"
                    
                    // Get the latest version of the artifact
                    def latestVersion = sh(returnStdout: true, script: "curl -s -u ${nexusAuth} ${searchUrl} | jq -r '.items[0].version'")
                    
                    // Construct the Nexus artifact URL for the latest version
                    def artifactUrl = "${NEXUS_URL}/repository/${repository}/${groupId.replace('.', '/')}/${artifactId}/${latestVersion}/${artifactId}-${latestVersion}.jar"
                    
                    // Pull the latest artifact using curl
                    sh "curl -u ${nexusAuth} -o ${artifactId}-${latestVersion}.jar ${artifactUrl}"
                }
            }
        }
    }
}
