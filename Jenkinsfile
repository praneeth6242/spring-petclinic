pipeline{ 
   agent { label 'JAVA_NEWNODE' } 
   triggers { pollSCM ('* * * * *') } 
    parameters {
        choice(name: 'MAVEN_GOAL', choices: ['package', 'install', 'clean'], description: 'Maven Goal')
    }
   stages {
      stage('clone') {
         steps {
             git url: 'https://github.com/praneeth6242/spring-petclinic.git',
              branch: 'declerative' 
         } 
      } 
	   stage ('Artifactory configuration') {
            steps {
                rtServer (
                    id: "ARTIFACTORY_SERVER",
                    url: 'https://praneeth6242.jfrog.io/artifactory',
                    credentialsId: 'JFROG'
                )

                rtMavenDeployer (
                    id: "MAVEN_DEPLOYER",
                    serverId: "ARTIFACTORY_SERVER",
                    releaseRepo: 'libs-release',
                    snapshotRepo: 'libs-snapshot'
                )

                rtMavenResolver (
                    id: "MAVEN_RESOLVER",
                    serverId: "ARTIFACTORY_SERVER",
                    releaseRepo:  'libs-release',
                    snapshotRepo: 'libs-snapshot'
                )
            }
        }

        stage ('rt Maven') {
		    tools {
              jdk 'JDK_17'
			  }
            steps {
                rtMavenRun (
                    tool: 'MAVEN-1', // Tool name from Jenkins configuration
                    pom: 'pom.xml',
                    goals: 'clean install',
                    deployerId: "MAVEN_DEPLOYER",
                    resolverId: "MAVEN_RESOLVER"
                )
            }
        }

        stage ('Publish build info') {
            steps {
                rtPublishBuildInfo (
                    serverId: "ARTIFACTORY_SERVER"
                )
            }
        }
        stage('SonarQube analysis') { 
         tools {
            jdk 'JDK_17'
            maven 'MAVEN-1'
         }
         steps('SonarQube analysis') {
            withSonarQubeEnv('THANVI_SPC') {
            sh 'mvn clean package sonar:sonar -Dsonar.organization=thanvispc -Dsonar.projectKey=thanvispc_spcjenkins'
            sh 'echo $RUN_ARTIFACTS_DISPLAY_URL'  
            sh 'echo $BUILD_URL' 
            sh 'echo $LATEST_ARTIFACT'
         } 
      }
      }
       stage("Quality Gate") {
         steps {
                timeout(time: 1, unit: 'HOURS') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true
                }
            }
        }
   }
   }