pipeline{ 
   agent { label 'JAVA_NODE1' } 
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
      stage('build') { 
         tools {
            jdk 'JDK_17'
            maven 'MAVEN'
         }
         steps {
            sh "mvn ${params.MAVEN_GOAL}"
         }
      } 
      stage ('postbuild') {
         steps {
           archiveArtifacts artifacts:  '**/target/spring-petclinic-3.0.0-SNAPSHOT.jar',
                         onlyIfSuccessful: true
            junit testResults: '**/surefire-reports/TEST-*.xml'
            stash name: 'spring',
                  includes:  '**/target/spring-petclinic-3.0.0-SNAPSHOT.jar'
      }
      }
   } 
   post {
      always {
         mail subject: 'the build ${JOB_NAME} with ${BUILD_ID} is build',
              body: 'build is either success or fail',
              from: 'praneeth@gmail.com',
              to: 'poorna@gmail.com'
      }
   }
}
