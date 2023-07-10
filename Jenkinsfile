pipeline {
    agent any
   // environment {
     //   NEXUS_URL = "65.2.135.243:8081"  
    //    NEXUS_USERNAME = 'admin'  
    //    NEXUS_PASSWORD = 'nexus123'  
    //}
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
       stage('Pull Artifacts') {
            steps {
                script {
                    def nexusUrl = '65.2.135.243:8081'
                    def nexusRepository = 'maven-snapshots'
                    def groupId = 'com.mkyong'
                    def artifactId = 'counterwebapp'
                    def version = '1.0-SNAPSHOT'
                    def classifier = ''
                    def s3Bucket = 'nexusartifactss3'
                    def awsRegion = 'ap-south-1'
                    
                    def artifactUrl = "${nexusUrl}/repository/${nexusRepository}/${groupId}/${artifactId}/${version}/${artifactId}-${version}-${classifier}.war"
                    
                    // Download artifact from Nexus
                    sh "curl -O ${artifactUrl}"
                    
                    // Configure AWS CLI
                    sh "aws configure set aws_access_key_id AKIASXYFD7RZ5NJLVD2T"
                    sh "aws configure set aws_secret_access_key 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"
                    sh "aws configure set region ${awsRegion}"
                    
                    // Upload artifact to S3 bucket
                    sh "aws s3 cp ${artifactId}-${version}-${classifier}.jar s3://${s3Bucket}/${groupId}/${artifactId}/${version}/${artifactId}-${version}-${classifier}.jar"
                  }
            }
        }
    }
}
