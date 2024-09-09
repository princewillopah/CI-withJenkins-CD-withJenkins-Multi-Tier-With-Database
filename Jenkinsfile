pipeline {
    agent any

    tools {
        // jdk 'jdk17'
        maven 'maven3'
    }
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }
    stages {
         stage('Clean Workspace'){
             steps{
                 cleanWs()
             }
         }
         stage('Checkout from Git'){
             steps{
                 git credentialsId: 'github-cred', url: 'https://github.com/princewillopah/CI-withJenkins-CD-withJenkins-Multi-Tier-With-Database.git'
             }
         }

        stage('Compile ...') {
             steps {
                sh "mvn compile"
             }
        }

        stage('Test') {
            steps {
                sh "mvn test  -DskipTests=true"  // "DskipTests=true" will make the stage be skipped
            }
        }

        //  stage('File System Scan') {
        //     steps {
        //         sh "trivy fs --format table -o trivy-fs-report.html ."
        //     }
        //  }

         stage('SonarQube Analsyis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=MultiTier -Dsonar.projectKey=MultiTier \
                    -Dsonar.java.binaries=. '''
                }
            }
         }
        stage('Build') {
            steps {
                sh "mvn package  -DskipTests=true"  // -DskipTests=true will avoid the test during build
            }
         }

         stage('Publish Artifact To Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'settings-xml-id', jdk: '', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy -DskipTests=true"   // -DskipTests=true will avoid the test during deploy
                }
            }
         }

             stage('Build & Tag Docker Image') {
                steps {
                    script {
                        withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker build -t princewillopah/multitier:latest ."
                        }
                    }
                }
             }

             stage('Docker Image Scan') {
                steps {
                sh "trivy image --format table -o trivy-image-report.html princewillopah/multitier:latest"
                }
             }

             stage('Push Docker Image') {
                steps {
                    script {
                        withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                            sh "docker push princewillopah/multitier:latest"
                        }
                    }
                }
             }


             stage('Deploy To Kubernetes') {
                steps {
                    withKubeConfig(caCertificate: '', clusterName: 'princewill_proj-cluster', contextName: '', credentialsId: 'eks-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://45CC2D9161D68FCF94CD8F626275B1B3.gr7.eu-north-1.eks.amazonaws.com') {
                        sh 'kubectl delete --all pods -n webapps'
                         sh "kubectl apply -f ds.yaml -n webapps"
                         sleep 30
                    }
                }
             }
             stage('Verify the Deployment') {
                steps {
                    withKubeConfig(caCertificate: '', clusterName: 'princewill_proj-cluster', contextName: '', credentialsId: 'eks-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://45CC2D9161D68FCF94CD8F626275B1B3.gr7.eu-north-1.eks.amazonaws.com') {
                        sh 'kubectl get pods -n webapps'
                         sh "kubectl get svc -n webapps"
                    }
                }
             }


    } // end stages

    
    // post {
    //     always {
    //         script {
    //         def jobName = env.JOB_NAME
    //         def buildNumber = env.BUILD_NUMBER
    //         def pipelineStatus = currentBuild.result ?: 'UNKNOWN'
    //         def bannerColor = pipelineStatus.toUpperCase() == 'SUCCESS' ? 'green' : 'red'
    //         def body = """
    //         <html>
    //             <body>
    //                 <div style="border: 4px solid ${bannerColor}; padding:10px;">
    //                     <h2>${jobName} - Build ${buildNumber}</h2>
    //                     <div style="background-color: ${bannerColor}; padding:10px;">
    //                         <h3 style="color: white;">Pipeline Status:${pipelineStatus.toUpperCase()}</h3>
    //                     </div>
    //                     <p>Check the <a href="${BUILD_URL}">console output</a>.</p>
    //                 </div>
    //             </body>
    //         </html>
    //         """
    //         emailext (
    //             subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus.toUpperCase()}",
    //             body: body,
    //             to: 'princewillopah.dev@gmail.com',
    //             from: 'jenkins@example.com',
    //             replyTo: 'jenkins@example.com',
    //             mimeType: 'text/html',
    //             attachmentsPattern: 'trivy-image-report.html'
    //             )
    //         }
    //     }
    // }
}