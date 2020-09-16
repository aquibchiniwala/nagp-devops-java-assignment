pipeline {
    agent any

    parameters {
        string(name: 'dtr', defaultValue: 'dtr.nagarro.com:443');
        string(name: 'userName', defaultValue: 'machiniwala');
    }

    tools {
        maven 'Maven3'
        jdk 'Java'
    }
    options {
        timestamps()

        timeout(time: 1, unit: 'HOURS')

        skipDefaultCheckout()

        buildDiscarder(logRotator(daysToKeepStr: '10', numToKeepStr: '10'))

        disableConcurrentBuilds()
    }
    stages {
        stage('Checkout') {
            steps {
                echo "Checkout"
                checkout scm
            }
        }
        stage('Build') {
            steps {
                echo "Build"
                bat "mvn clean install"
            }
        }
        stage('Unit Testing') {
            steps {
                echo "Unit Testing"
                bat "mvn test"
            }
        }
        stage ('Sonar Analysis') {
            steps {
                echo "Sonar Analysis"
                withSonarQubeEnv("Test_Sonar")
                {
                    bat "mvn sonar:sonar"
                }
            }
        }
        stage ('Upload to Artifactory') {
            steps {
                echo "Upload to Artifactory"
                rtMavenDeployer (
                    id: 'tester',
                    serverId: '123456789@artifactory',
                    releaseRepo: 'CI-Automation-JAVA',
                    snapshotRepo: 'CI-Automation-JAVA'
                )
                rtMavenRun (
                    pom: 'pom.xml',
                    goals: 'clean install',
                    deployerId: 'tester'
                )
                rtPublishBuildInfo (
                    serverId: '123456789@artifactory'
                )
            }
       }
        stage('Docker Image') {
            steps {
                echo "Build Docker Image "
                bat "docker build -t ${params.dtr}/i_${params.userName}_master:${BUILD_NUMBER} --no-cache -f Dockerfile ."
            }
        }
		stage ('Push to DTR') {
            steps {
			echo "Push Docker Image"
                	bat "docker push ${params.dtr}/i_${params.userName}_master:${BUILD_NUMBER}"
            }
        }
        
        stage('Docker deployment') { 
            steps {
                echo 'Run Docker Image'
                bat "docker run --name c_${params.userName}_master -d -p 6000:8080 ${params.dtr}/i_${params.userName}_master:${BUILD_NUMBER}"
            }
        }
        stage ('Helm Chart Deployment') {
        	steps {
                echo 'Deploy Helm Chart'
                bat "kubectl create ns ${params.userName}-master-${BUILD_NUMBER}"
        		bat "helm install helm-chart-master helm-charts --set image=${params.dtr}/i_${params.userName}_master:${BUILD_NUMBER} --set nodeport=30157 --namespace=${params.userName}-master-${BUILD_NUMBER}"
        	    }
        	}
        }
    }
}