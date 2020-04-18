pipeline {
    agent any
    
    tools {
          maven 'maven'
        }

    stages {
        stage("build & SonarQube analysis") {
            agent any
            steps {
              withSonarQubeEnv('sonar') {
                slackSend channel: '#devops-casestudy-group7', message: 'slackSend "started Pipeline-by-JenkinsFile"', teamDomain: 'devops-casestudy', tokenCredentialId: 'slack-key'  
                sh 'mvn clean package sonar:sonar -Dsonar.host.url=http://13.88.184.36:9000/ -Dsonar.login=admin -Dsonar.password=devops123 -Dsonar.sources=. -Dsonar.tests=. -Dsonar.inclusions=**/test/java/servlet/createpage_junit.java -Dsonar.test.exclusions=**/test/java/servlet/createpage_junit.java'
              }
            }
          }
          stage("Quality Gate") {
            steps {
              timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true
              }
            }
          }
       
         stage('Artifactory'){
            steps {
                script{
                    node {
                            // Get Artifactory server instance, defined in the Artifactory Plugin administration page.
                            def server = Artifactory.server "artifactory"
                            // Create an Artifactory Maven instance.
                            def rtMaven = Artifactory.newMavenBuild()
                            def buildInfo

                         rtMaven.tool = "maven"        

                            stage('Artifactory configuration') {
                                echo 'Configuring Artifactory'
                                // Tool name from Jenkins configuration
                                rtMaven.tool = "maven"
                                // Set Artifactory repositories for dependencies resolution and artifacts deployment.
                                rtMaven.deployer releaseRepo:'libs-release-local', snapshotRepo:'libs-snapshot-local', server: server
                                rtMaven.resolver releaseRepo:'libs-release', snapshotRepo:'libs-snapshot', server: server
                            }

                            stage('Maven build') {
                                buildInfo = rtMaven.run pom: 'pom.xml', goals: 'clean install'
                            }

                            stage('Publish build info') {
                                server.publishBuildInfo buildInfo
                            }
                          
                    }  
                }
            }
        }
        
        stage('Deploy to QA ') {
            steps {
                echo 'Deploying to QA ....'          
                sh 'mvn package'
                deploy adapters: [tomcat7(credentialsId: 'tomcat', path: '', url: 'http://18.191.231.46:8080/')], contextPath: '/QAWebapp', onFailure: false, war: '**/*.war'
            }
        }
        stage('Functional Testing') {
            steps {
                echo 'Testing..'
                sh 'mvn -f functionaltest/pom.xml test'
                publishHTML (target : [allowMissing: false,
                     alwaysLinkToLastBuild: true,
                     keepAll: true,
                     reportDir: 'functionaltest/target/surefire-reports',
                     reportFiles: 'index.html',
                     reportName: 'My Reports',
                     reportTitles: 'The Report'])
            }
        }
        
        stage('Perfomance Testing') {
            steps {
                echo 'Performance Testing'
                blazeMeterTest credentialsId: 'Blazemeter-Updated', testId: '7914994.taurus', workspaceId: '475894'
                //blazeMeterTest credentialsId: 'Blazemeter-Updated', testId: '7895696.taurus', workspaceId: '475895'
                //blazeMeterTest credentialsId: 'blazemeterdemo', testId: '7907341.taurus', workspaceId: '475906'
            }
        }
        stage('Deploy to Prod') {
            steps {
                echo 'Deploying to Prod ....'
                sh 'mvn package'
                deploy adapters: [tomcat7(credentialsId: 'tomcat', path: '', url: 'http://18.216.230.83:8080/')], contextPath: '/ProdWebapp', onFailure: false, war: '**/*.war'
            }
        }
        stage('Acceptance Testing') {
            steps {
                echo 'Testing..'
                sh 'mvn -f Acceptancetest/pom.xml test'
                publishHTML (target : [allowMissing: false,
                     alwaysLinkToLastBuild: true,
                     keepAll: true,
                     reportDir: 'Acceptancetest/target/surefire-reports',
                     reportFiles: 'index.html',
                     reportName: 'My Reports',
                     reportTitles: 'The Report'])
            }
        }
        
    }
}
