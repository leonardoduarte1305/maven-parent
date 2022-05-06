pipeline {
	agent any
	tools {
		maven 'maven-3'
		jdk 'jdk-11'
	}
	options {
		disableConcurrentBuilds()
		skipStagesAfterUnstable()
		timestamps()
	}
	environment {
		VERSION = "${Versioner.generateVersionFromHistory(this, BRANCH_NAME, '3')}"
		MAVEN_OPTS = "-Djansi.force=true"
		MAVEN = "mvn -s settings.xml -B -Dstyle.color=always -Dmaven.test.redirectTestOutputToFile=false"
	}
	stages {
		stage('Init') {
			steps {
				script {
					currentBuild.description = "${VERSION}"
				}
				ansiColor('xterm') {}
				configFileProvider([configFile(fileId: 'global-settings.xml', targetLocation: 'settings.xml')]) {}
				sh "${MAVEN} assembly:single versions:set versions:set-property -DnewVersion=${VERSION}"
			}
		}
		stage("Build") {
			steps {
				sh "${MAVEN} clean verify"
			}
		}
		stage('Git Tag') {
			when {
				branch 'master'
			}
			steps {
				sh "git tag ${VERSION}"
				sh "git push origin ${VERSION}"
			}
		}
		stage("Deploy") {
			steps {
				sh "${MAVEN} assembly:single deploy:deploy"
			}
		}
	}
}