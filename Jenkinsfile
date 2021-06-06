pipeline{
    agent any
    
    stages{
        stage("clone"){
            steps{
                git "https://github.com/prashpb/batch10.git"
            }
        }
        stage("build"){
            steps{
                sh "mvn clean package surefire-report:report"
            }
        }
        stage("Publish report"){
            steps{
                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'target/site', reportFiles: 'surefire-report.html', reportName: 'JUnit Test Report', reportTitles: ''])
            }
        }
        stage("Sonar"){
            steps{
                withCredentials([usernamePassword(credentialsId: 'sonar', passwordVariable: 'sonarpass', usernameVariable: 'sonar')]) {
                    sh 'mvn sonar:sonar -Dsonar.projectKey=batch10 -Dsonar.login=admin -Dsonar.password=${sonarpass} -Dsonar.host.url=http://35.240.193.38:9000'
                }
                
            }
        }
        stage("Docker Build"){
            steps{
                sh "docker build -t prashpb/bootcamp:1 ."
            }
        }
        stage("Docker Push"){
            steps{
                withCredentials([usernamePassword(credentialsId: 'docker', passwordVariable: 'dockerpass', usernameVariable: 'docker')]) {
                    sh "docker login -u prashpb -p ${dockerpass}"
                    sh "docker push prashpb/bootcamp:1"
                }
                
            }
        }
        stage("deploy app"){
            steps{
                ansiblePlaybook credentialsId: 'jenkins', disableHostKeyChecking: true, installation: 'ansible', inventory: 'hosts', playbook: 'deploy_ansible.yaml'
                
            }
        }
    }
    post{
        success{
            emailext body: 'Jenkins Build logs ${BUILD_URL}', subject: 'Jenkins Build Success', to: 'livtorock@gmail.com'
        }
        failure{
            emailext body: 'Jenkins Build logs ${BUILD_URL}', subject: 'Jenkins Build Failure', to: 'livtorock@gmail.com'
        }
    }
}