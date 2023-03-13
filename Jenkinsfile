
pipeline{ 
   agent { label 'JAVA_NODE1' } 
    scm {
        git {
            remote {
                github('https://github.com/praneethkunka/spring-petclinic.git')
                refspec('+refs/pull/*:refs/remotes/origin/pr/*')
            }
            branch('{ghprbActualCommit}')
        }
    }
     triggers {
        githubPullRequest {
            admin('praneethkunka')
            cron('* * * * *')
           
            extensions {
                commitStatus {
                    context('deploy to staging site')
                    triggeredStatus('starting deployment to staging site...')
                    startedStatus('deploying to staging site...')
                    statusUrl('http://mystatussite.com/prs')
                    completedStatus('SUCCESS', 'All is well')
                    completedStatus('FAILURE', 'Something went wrong. Investigate!')
                    completedStatus('PENDING', 'still in progress...')
                    completedStatus('ERROR', 'Something went really wrong. Investigate!')
                }
            }
        }
    } 
    parameters {
        choice(name: 'MAVEN_GOAL', choices: ['package', 'install', 'clean'], description: 'Maven Goal')
    }
   stages {
      stage('clone') {
         steps {
             git url: 'https://github.com/praneeth6242/spring-petclinic.git',
              branch: 'main' 
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
                    tool: 'MAVEN', // Tool name from Jenkins configuration
                    pom: 'maven-examples/maven-example/pom.xml',
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
           steps('SonarQube analysis') {
             withSonarQubeEnv('THANVI_SPC') {
             sh 'mvn clean package sonar:sonar -Dsonar.organization=thanvispc'
         } 
      }
      }
   } 
}
   