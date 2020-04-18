pipeline {
    agent any
    
    tools {
          maven 'maven'
        }

    stages {
        stage("package") {
            steps {
               
                slackSend channel: '#devops-casestudy-group7', message: 'slackSend "started Pipeline-by-JenkinsFile"', teamDomain: 'devops-casestudy', tokenCredentialId: 'slack-key'  
                //sh 'mvn clean package' 
                sh 'mvn -f pom.xml clean package'
            }
          }
         stage("Ansible") {
            steps {
                sshPublisher(publishers: [sshPublisherDesc(configName: 'Ansible', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '''ansible-playbook -i /opt/docker/hosts /opt/docker/docker-create-push-webapp.yml --limit localhost
                ansible-playbook -i /opt/docker/hosts /opt/docker/docker-pull-run-webapp.yml --limit 23.99.10.237''', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '//opt//docker', remoteDirectorySDF: false, removePrefix: 'webapp/target/', sourceFiles: 'webapp/target/*.war')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
        }
      }
    }
}
