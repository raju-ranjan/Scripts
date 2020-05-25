pipeline{
	agent any
	tools {
      // Install the Maven version configured as "M3" and add it to the path.
      maven "maven"
   }
	stages{
		stage ('SCM Clone'){
			steps{
				git branch: 'master', url: "https://github.com/vikash21kumar/hello-app.git"
			}
			
			
		}

		stage ('Artifactory configuration') {
            steps {
                rtServer (
                    id: "txdevops",
                    url: "https://txdevopsbootcamp.jfrog.io/txdevopsbootcamp",
                    credentialsId: "artifactory-txdevops"
                )

                rtMavenDeployer (
                    id: "MAVEN_DEPLOYER",
                    serverId: "txdevops",
                    releaseRepo: "libs-release-local",
                    snapshotRepo: "libs-snapshot-local"
                )

                rtMavenResolver (
                    id: "MAVEN_RESOLVER",
                    serverId: "txdevops",
                    releaseRepo: "libs-release",
                    snapshotRepo: "libs-snapshot"
                )
                
            }
        }

		stage ('Exec Maven') {
            steps {
                slackSend channel: '#cicd', message: "Build ${BUILD_NUMBER} for hello-world App Started with commit"
                rtMavenRun (
                                tool: "maven", // Tool name from Jenkins configuration
                                pom: 'pom.xml',
                                goals: 'clean install ',
                                deployerId: "MAVEN_DEPLOYER",
                                resolverId: "MAVEN_RESOLVER"
                            )
                    }
            post {
                always {
                    slackSend channel: '#cicd', message: "Build Completed for commit"
                    //jiraSendBuildInfo branch: 'master', site: 'txdevopsbootcamp.atlassian.net'
                    //jiraComment body: "Build 'env.BUILD_NUMBER' completed for commit (${GIT_COMMIT})", issueKey: 'DEMO-1'
                   
                    
                    }
                }
        }
		stage ('Ansible Deployment to Remote Server') {
           
            steps {
                sshPublisher(publishers: [sshPublisherDesc(configName: 'Ansible', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '''ansible-playbook -i /opt/docker/hosts /opt/docker/docker-create-push-webapp.yml --limit localhost

				ansible-playbook -i /opt/docker/hosts /opt/docker/docker-pull-run-webapp.yml --limit 40.87.31.226''', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '//opt//docker', remoteDirectorySDF: false, removePrefix: 'webapp/target', sourceFiles: 'webapp/target/*.war')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
            }
        
        }



	}



}
