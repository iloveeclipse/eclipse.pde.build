pipeline {
	options {
		timeout(time: 40, unit: 'MINUTES')
		buildDiscarder(logRotator(numToKeepStr:'5'))
	}
	agent {
		label "centos-latest"
	}
	tools {
		maven 'apache-maven-latest'
		jdk 'openjdk-jdk11-latest'
	}
	stages {
		stage('Build') {
			steps {
				wrap([$class: 'Xvnc', useXauthority: true]) {
					sh """
					mvn clean verify --batch-mode --fail-at-end \
						-Pbuild-individual-bundles -Pbree-libs -Papi-check \
						-Dcompare-version-with-baselines.skip=false \
						-Dproject.build.sourceEncoding=UTF-8 
					"""
				}
			}
			post {
				always {
					publishIssues issues:[scanForIssues(tool: java()), scanForIssues(tool: mavenConsole())]
				}
			}
		}
		stage('Check freeze period') {
			steps {
				sh "wget https://download.eclipse.org/eclipse/relengScripts/scripts/verifyFreezePeriod.sh"
				sh "chmod +x verifyFreezePeriod.sh"
				withCredentials([string(credentialsId: 'google-api-key', variable: 'GOOGLE_API_KEY')]) {
					sh './verifyFreezePeriod.sh'
				}
			}
		}
	}
}
