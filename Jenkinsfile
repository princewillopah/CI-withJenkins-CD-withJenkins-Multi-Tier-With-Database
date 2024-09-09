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
                sh "mvn package"  
            }
         }

         stage('Publish Artifact To Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'settings-xml-id', jdk: '', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy"  
                }
            }
         }

        // stage('Quality Gate') {
        //     steps {
        //         script {
        //             waitForQualityGate abortPipeline: false, credentialsId: 'SonarQube-Credentials'
        //         }
        //     }
        // }

        //  stage('Build') {
        //     steps {
        //         sh "mvn package"  
        //     }
        //  }

     

        //      stage('Build & Tag Docker Image') {
        //         steps {
        //             script {
        //                 withDockerRegistry(credentialsId: 'DockerHub-Credentials', toolName: 'my-docker') {
        //                 sh "docker build -t princewillopah/boardgame:latest ."
        //                 }
        //             }
        //         }
        //      }

        //      stage('Docker Image Scan') {
        //         steps {
        //         sh "trivy image --format table -o trivy-image-report.html princewillopah/boardgame:latest"
        //         }
        //      }

        //      stage('Push Docker Image') {
        //         steps {
        //             script {
        //                 withDockerRegistry(credentialsId: 'DockerHub-Credentials', toolName: 'my-docker') {
        //                     sh "docker push princewillopah/boardgame:latest"
        //                 }
        //             }
        //         }
        //      }
        //      stage('Deploy To Kubernetes') {
        //         steps {
        //             withKubeConfig(caCertificate: '', clusterName: 'boodgame-cluster.eu-north-1.eksctl.io', contextName: '', credentialsId: 'eks-credentials', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://AF38C7B9100F8B046960FE28099DAC95.yl4.eu-north-1.eks.amazonaws.com') {
        //                 sh 'kubectl delete --all pods -n webapps'
        //                  sh "kubectl apply -f deployment-service.yaml"
        //             }
        //         }
        //      }

        //      stage('Verify the Deployment') {
        //         steps {
        //             withKubeConfig(caCertificate: '', clusterName: 'boodgame-cluster.eu-north-1.eksctl.io', contextName: '', credentialsId: 'eks-credentials', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://AF38C7B9100F8B046960FE28099DAC95.yl4.eu-north-1.eks.amazonaws.com') {
        //                  sh "kubectl get pods -n webapps"
        //                  sh "kubectl get svc -n webapps"
        //             }

        //         }
        //      }


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