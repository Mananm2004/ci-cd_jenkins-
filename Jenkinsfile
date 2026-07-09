//testing github webhook 
pipeline{
    agent any 
    tools{
        maven 'Maven'
    }
    environment {
                    DOCKER_REPO = "mananm2004/manan"
    }
    stages{

        stage('incrementing the version'){
             steps {
                script {
                    echo 'incrementing app version...'
                    sh 'mvn build-helper:parse-version versions:set \
                        -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} \
                        versions:commit'
                    def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
                    def version = matcher[0][1]
                    env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                }
        }
        }
        stage('Build jar'){
            steps{
                script{
                    echo "building the jar for branchname: ${env.BRANCH_NAME} and build number: ${env.BUILD_NUMBER}"
                    sh 'mvn package'
                }
            }
        }
        stage('Build and push the docker image to dockerhub'){
            steps{
                script{
                    echo 'Building docker image...'
                    sh "docker build -t ${DOCKER_REPO}:$IMAGE_NAME ."
                   
                    withCredentials([
                        usernamePassword(
                            credentialsId: 'docker-hub-credentials',
                            usernameVariable: 'USER',
                            passwordVariable: 'PASS'
                        )
                    ]) {
                        sh 'echo "$PASS" | docker login -u "$USER" --password-stdin'
                        sh "docker push ${DOCKER_REPO}:${IMAGE_NAME}"
                    }

                }
            }
        }
        stage('Deploy to ec2'){
            steps{
                script{
                    echo 'Deploying to ec2...'
                    def  shellCmd = "bash  ./server-cmds.sh ${DOCKER_REPO}:${IMAGE_NAME}"
                    def ec2Instance = "ec2-user@13.61.174.182"
                    sshagent(['ec2-ssh-key']) {
                        sh "scp -o StrictHostKeyChecking=no server-cmds.sh  ${ec2Instance}:/home/ec2-user/server-cmds.sh"
                        sh "scp -o StrictHostKeyChecking=no docker-compose.yaml ${ec2Instance}:/home/ec2-user"
                        sh "ssh -o StrictHostKeyChecking=no ${ec2Instance} ${shellCmd}"
                    }

                }
            }
        }
        stage('Commit version update') {
                steps {
                    script {
                        sshagent(['Manan_github']) {
                            sh '''
                                git config user.email "jenkins@example.com"
                                git config user.name "Jenkins"

                                git add .
                                git commit -m "Incremented the version" || true
                                git push origin HEAD:master
                            '''
                        }
                    }
                }
    }
    }
}
