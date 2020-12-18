pipeline {
	agent none
	options { skipDefaultCheckout(true)}

	environment {
	  MAVEN_TOOL="apache-maven-3.5.3(default)"
	  ARTIFACTORY_URL="https://deepakpalartifactory.jfrog.io/ui/admin/repositories/local"
	  ARTIFACTORY_REPO="maven-repo"
	}

	stages {
    
      stage("Provisioning agent")

      stages {

        stage("Prepare") {
          steps {
            checkout scm
            script {
              artifactoryServer = Artifactory.newServer url: ARTIFACTORY_URL, username: "${env.ARTIFACTORY_ACCESS_USR}", password: "{env.ARTIFACTORY_ACCESS_PSW}"
              artifactoryMaven = Artifactory.newMavenBuild()
              artifactoryMaven.tool = MAVEN_TOOL
              artifactoryMaven.resolver releaseRepo: ARTIFACTORY_REPO, snapshotRepo: ARTIFACTORY_REPO, server: artifactoryServer
            }
          }
        }

        stage("Build") {
          steps {
            script{
              artifactoryMaven.run pom: 'pom.xml', goals: 'clean install -DskipTests=true'
            }
          }
        }

        stage("Unit Tests") {
          when { not { expression { return params.skipTest } } }
          steps {
            script{
              artifactoryMaven.run pom: 'pom.xml', goals: 'test'
              junit '**/target/surefire-reports/TEST-*.xml'
            }
          }
        }

        stage('Push to Artifactory (opcofr-maven-local-dev)') {
          steps{
            script{
              def mvn_pom = readMavenPom file: 'pom.xml'
              def path = "opcofr-maven-local-dev/" + mvn_pom.groupId.replaceAll("\\.","/")

              uploadSpec = """{
                "files": [
                  { "pattern": "pom.xml",    "target": "${path}/JEBO001M/${mvn_pom.version}/JEBO001M-${mvn_pom.version}.pom" },
                  { "pattern": "**/JEBO001M-${mvn_pom.version}.jar",    "target": "${path}/JEBO001M/${mvn_pom.version}/JEBO001M-${mvn_pom.version}.jar" },
                  { "pattern": "**/JEBO001M-${mvn_pom.version}-batch.jar",    "target": "${path}/JEBO001M/${mvn_pom.version}/JEBO001M-${mvn_pom.version}-batch.tar" }
                  ]
              }"""
              buildInfo = artifactoryServer.upload uploadSpec
              artifactoryServer.publishBuildInfo buildInfo
            }
          }
        } 
      }

	}
}
