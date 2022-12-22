pipeline{

    agent any 

    tools{
        maven "Maven"
    }

    stages{

        stage("Maven Packaging"){
          steps{
           sh 'mvn package'
          }
        }

        stage("Build Image") {
            steps{
                sh 'docker build -t java-maven-app:$BUILD_NUMBER .'
            }
        }

        

        stage("Logging Into ECR"){
            steps{
                sh 'aws ecr get-login-password --region ap-northeast-1 | docker login --username AWS --password-stdin 266454083192.dkr.ecr.ap-northeast-1.amazonaws.com'
            } 
        }

        stage("Pushing Image To ECR"){
            steps{
                sh 'docker tag java-maven-app:$BUILD_NUMBER 266454083192.dkr.ecr.ap-northeast-1.amazonaws.com/maven-app:$BUILD_NUMBER'
                sh 'docker push 266454083192.dkr.ecr.ap-northeast-1.amazonaws.com/maven-app:$BUILD_NUMBER'
            }
        }
        
        stage("Logging into Helm repository "){
            steps{
                sh 'aws ecr get-login-password --region ap-northeast-1 | helm registry login --username AWS --password-stdin 266454083192.dkr.ecr.ap-northeast-1.amazonaws.com'
            }
        }

        stage("Update image tag in helm value file"){
            steps{
                sh "sed -i ' s/appVersion.*/appVersion: '$BUILD_NUMBER'/ ' ./HELM-CHART/values.yaml "
                sh "cat ./HELM-CHART/values.yaml "

            }
        }

        stage("pushing helm package into ECR"){
            steps{
                sh "sed -i ' s/version.*/version: '$BUILD_NUMBER'/ ' ./HELM-CHART/Chart.yaml "
                sh "cat ./HELM-CHART/Chart.yaml "
                sh "helm package HELM-CHART "
                sh "helm push helm-repo-'$BUILD_NUMBER'.tgz oci://266454083192.dkr.ecr.ap-northeast-1.amazonaws.com "
            }
        }    
    }

    post{ 
        success {
            withCredentials([usernamePassword(credentialsId: "git-auth", passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                        sh "git config user.email abbashussain.x@gmail.com"
                        sh "git config user.name abbas"
                        sh "git add . "
                        sh "git commit -m 'trigger build' "
                        sh 'git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/abbashussainz/CI-CD-Argo-CD.git'
                    }

        }
    }
}