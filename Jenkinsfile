def CONTAINER_NAME="jenkins-pipeline"
def CONTAINER_TAG="latest"
def DOCKER_HUB_USER="gajulank"
def HTTP_PORT="8090"

node {

    stage('Initialize'){
        def dockerHome = tool 'myDocker'
        def mavenHome  = tool 'myMaven'
        env.PATH = "${dockerHome}/bin:${mavenHome}/bin:${env.PATH}"
    }

    stage('Checkout') {
        checkout scm
    }

    stage('Build'){
        sh "mvn clean install"
    }
stage('Sonar'){
        try {
            sh "mvn sonar:sonar"
        } catch(error){
            echo "The sonar server could not be reached ${error}"
        }
     }
  

    stage('Image Build'){
        imageBuild(CONTAINER_NAME, CONTAINER_TAG)
    }

    stage('Push to Docker Registry'){
        withCredentials([usernamePassword(credentialsId: 'dockerHubAccount', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
            pushToImage(CONTAINER_NAME, CONTAINER_TAG, USERNAME, PASSWORD)
        }
    }

    stage('Deploy App in DEV'){
        runApp(CONTAINER_NAME, CONTAINER_TAG, DOCKER_HUB_USER, HTTP_PORT)
    }
    stage('Smoke Test in DEV'){
        sh "ssh -o StrictHostKeyChecking=no -i /root/AWSDemo.pem ec2-user@35.175.251.180 \"sudo sh /tmp/dev.sh\""
    }
    stage('Deploy App in QA'){
        runAppQA(CONTAINER_NAME, CONTAINER_TAG, DOCKER_HUB_USER, HTTP_PORT)
    }
    stage('Smoke Test in QA'){
        sh "ssh -o StrictHostKeyChecking=no -i /root/AWSDemo.pem ec2-user@34.207.133.119 \"sudo sh /tmp/QA.sh\""
    }
    stage('Deploy App in Staging'){
        runAppSTG(CONTAINER_NAME, CONTAINER_TAG, DOCKER_HUB_USER, HTTP_PORT)
    }
    stage('Smoke Test in STG'){
        sh "ssh -o StrictHostKeyChecking=no -i /root/AWSDemo.pem ec2-user@18.234.156.152 \"sudo sh /tmp/STG.sh\""
    }
}

def imagePrune(containerName){
    try {
        sh "docker image prune -f"
        sh "docker stop $containerName"
    } catch(error){}
}

def imageBuild(containerName, tag){
    sh "docker build -t $containerName:$tag  -t $containerName --pull --no-cache ."
    echo "Image build complete"
}

def pushToImage(containerName, tag, dockerUser, dockerPassword){
    sh "docker login -u $dockerUser -p $dockerPassword"
    sh "docker tag $containerName:$tag $dockerUser/$containerName:$tag"
    sh "docker push $dockerUser/$containerName:$tag"
    echo "Image push complete"
}

def runApp(containerName, tag, dockerHubUser, httpPort){
    sh "ssh -o StrictHostKeyChecking=no -i /root/AWSDemo.pem ec2-user@35.175.251.180 \"sudo docker stop $containerName;sudo docker pull $dockerHubUser/$containerName\""
 
    sh "ssh -o StrictHostKeyChecking=no -i /root/AWSDemo.pem ec2-user@35.175.251.180 \"sudo docker run -d --rm -p $httpPort:$httpPort --name $containerName $dockerHubUser/$containerName:$tag\""
    echo "Application started on port: ${httpPort} (http)"
}
def runAppQA(containerName, tag, dockerHubUser, httpPort){
    sh "ssh -o StrictHostKeyChecking=no -i /root/AWSDemo.pem ec2-user@34.207.133.119 \"sudo docker stop $containerName;sudo docker pull $dockerHubUser/$containerName\""
 
    sh "ssh -o StrictHostKeyChecking=no -i /root/AWSDemo.pem ec2-user@34.207.133.119 \"sudo docker run -d --rm -p $httpPort:$httpPort --name $containerName $dockerHubUser/$containerName:$tag\""
    echo "Application started on port: ${httpPort} (http)"
}
def runAppSTG(containerName, tag, dockerHubUser, httpPort){
    sh "ssh -o StrictHostKeyChecking=no -i /root/AWSDemo.pem ec2-user@18.234.156.152 \"sudo docker stop $containerName;sudo docker pull $dockerHubUser/$containerName\""
 
    sh "ssh -o StrictHostKeyChecking=no -i /root/AWSDemo.pem ec2-user@18.234.156.152 \"sudo docker run -d --rm -p $httpPort:$httpPort --name $containerName $dockerHubUser/$containerName:$tag\""
    echo "Application started on port: ${httpPort} (http)"
}
