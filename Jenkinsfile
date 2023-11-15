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

         stage('Build Frontend') {
            steps {
                dir('DevOps_Front') {
                    echo 'Installing dependencies...'
                    sh 'npm install'
                    echo 'Building Angular project...'
                    sh 'ng build'
                }
            }
            post {
                success {
                    echo 'Frontend build successful.'
                }
                failure {
                    echo 'Frontend build failed.'
                }
            }
        }

      
        

  stage('Deploy to Nexus') {
            steps {
                script {
                    def artifactFile = "DevOps_Backend/target/DevOps_Project-1.0.jar" // Replace with the actual artifact name pattern
                    nexusArtifactUploader(
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: "${NEXUS_IP}:${NEXUS_PORT}",
                        groupId: 'QA',
                        version: "${env.BUILD_ID}-${new Date().format('yyyyMMddHHmmss')}", // Correct timestamp format
                        repository: "${RELEASE_REPO}",
                        credentialsId: "${NEXUS_LOGIN}",
                        artifacts: [
                            [artifactId: 'DevOps_Project',
                             classifier: '',
                             file: artifactFile,
                             type: 'jar']
                        ]
                    )
                }
            }
        }


   


        stage('Build Docker Images') {
         steps {
            script {
             // Build and push backend image
             dir('DevOps_Backend') {
                 docker.build("mondherbha1999/devopsproject", "-f /var/lib/jenkins/workspace/Devops_project/DevOps_Backend/Dockerfile .")
             }

            // // Build and push frontend image
             // dir('DevOps_Front') {
                //     docker.build("hamzabouzidi/devopsfrontend", "-f /var/lib/jenkins/workspace/projetDevOps/DevOps_Front/Dockerfile .")
             // }
        }
     }
 }


         stage('Push image to Hub') {
    steps {
        script {
            withCredentials([string(credentialsId: 'docker-hub-credentials-id', variable: 'DOCKER_HUB_PASSWORD')]) {
                dir('DevOps_Backend') {
                    sh "docker login -u mondherbha1999 -p ${DOCKER_HUB_PASSWORD}"
                    sh "docker push mondherbha1999/devops_project"
                }
            }
        }
    }
}


        stage('Build and Deploy') {
            steps {
                script {
                    sh '/usr/bin/docker-compose -f /var/lib/jenkins/workspace/projetDevOps/docker-compose.yml up -d'
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
