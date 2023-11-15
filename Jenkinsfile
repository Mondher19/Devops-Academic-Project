pipeline {
    agent any

    environment {
        MVN_HOME = tool 'Maven' // Make sure 'Maven' is the name of the tool configured in Jenkins
        NODEJS_HOME = tool 'NodeJS' // Make sure 'NodeJS' is the name of the tool configured in Jenkins
        NEXUS_USER = 'admin'
        NEXUS_PASSWORD = 'admin'
        SNAP_REPO = 'Mondher_Nexus_Repo-snapshot'
        RELEASE_REPO = 'Mondher_Nexus_Repo'
        CENTRAL_REPO = 'Mondher_Nexus_Repo-central-repo'
        NEXUS_GRP_REPO = 'Mondher_Nexus_Repo-grp-repo'
        NEXUS_IP = '172.17.0.2'
        NEXUS_PORT = '8081'
        NEXUS_LOGIN = '485b967a-8f70-498c-8c77-b47207262286'
        // Dynamically assign 'version' or ensure it matches the version in pom.xml
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Unit Test') {
            steps {
                dir('DevOps_Backend') {
                    script {
                        sh "${MVN_HOME}/bin/mvn clean test"
                    }
                }
            }
        }

        stage('JaCoCo Results') {
            steps {
                script {
                    def jacocoReportPath = 'DevOps_Backend/target/site/jacoco'

                    publishHTML(
                        target: [
                            allowMissing: false,
                            alwaysLinkToLastBuild: true,
                            keepAll: true,
                            reportDir: jacocoReportPath,
                            reportFiles: 'index.html',
                            reportName: 'JaCoCo Code Report'
                        ]
                    )
                }
            }
        }  

       stage('Build Backend') {
            steps {
                dir('DevOps_Backend') {
                    script {
                        sh "${MVN_HOME}/bin/mvn clean package"
                    }
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                }
            }

             post {
        success {
            mail to: 'mondher.benhajammar@esprit.tn',
                 subject: "Build Backend",
                 body: "Build Backend succeeded."
        }
        failure {
            mail to: 'mondher.benhajammar@esprit.tn',
                 subject: "Build Backend",
                 body: "Build Backend failed."
        }
    }
        } 
        


  stage('SonarQube Analysis') {
    steps {
        script {
            // Checkout the source code from GitHub
            checkout scm
            
            def scannerHome = tool 'SonarQubeScanner'
            withSonarQubeEnv('SonarQube') {
                sh """
                    ${scannerHome}/bin/sonar-scanner \
                    -Dsonar.projectKey=Mondher_Devops \
                    -Dsonar.java.binaries=DevOps_Backend/target/classes
                    -Dsonar.coverage.jacoco.xmlReportPaths=DevOps_Backend/target/site/jacoco/jacoco.xml
                """
            }
        }
    }
}
   



        
        
    }

    post {
        always {
            cleanWs()
        }
    }
}
