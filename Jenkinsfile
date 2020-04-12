@Library(value = 'utilities')_
import hudson.Util;
pipeline {
    agent {
        kubernetes {
            label 'jnlp-npm-builder'
            defaultContainer 'builder'
        }
    }
    parameters {
		string(name: 'git_repo_source', defaultValue: 'node.git', description: 'Git repository from where we are going to checkout the code (master branch) and build a docker image. NB! The repository must contain a Dockerfile in the root')
        string(name: 'docker_repository', defaultValue: 'hfc/dev', description: 'The docker repository')
        string(name: 'registry_url', defaultValue: 'URL', description: 'Container Registry URL')
        string(name: 'git_repo_branch', defaultValue: 'dev', description: 'The branch to be checked out')
        string(name: 'releaseVersion', defaultValue: '3.0', description: 'release tag as per major release')
    }
    options {
        disableConcurrentBuilds()
        buildDiscarder(logRotator(daysToKeepStr: '7', artifactDaysToKeepStr: '7'))
        timeout(time: 1, unit: 'HOURS')
    }
    triggers {
        pollSCM('* * * * *')
    }
   stages {
        stage('Checkout git repo') {
            steps {
            	gitCheckout()
            }
        }
 	stage('Build,Scan and Push Docker image'){
            steps {
                script {
                     buildAndPushToAzureRegistry()
                }
            }
        }
   }
    post { 
        always {
           	mail (to: 'vikas@example.com',
            subject: "[HFC-DEVOPS] JOB: Frontend  CURRENT_STATUS: ${currentBuild.currentResult}",
		    body: "Changes:\n " + getChangeString() + "\n\n Check console output at: $BUILD_URL/console" + "\n")
			echo "Deleting docker images with name ${params.docker_repository} that can eat up disk space"
			sh "docker images | grep ${params.docker_repository} | awk '{print \$3}' | xargs docker rmi -f"
		}
    }
}


