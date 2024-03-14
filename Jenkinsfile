pipeline {
  agent any

tools {
    nodejs "Nodejs"
}
    environment {
       AWS_REGION = 'ap-south-1'
        AWS_ACCOUNT_ID = '211125384091'
        ECR_REPOSITORY = 'demo-jen'
        IMAGE_TAG = 'latest'
        SONAR_TOKEN = credentials('sonarqube')
        ECS_CLUSTER = 'jenkins-demo'
        ECS_SERVICE = 'jenkins-svc'
        DOCKER_IMG = 'mrlokidocs/angular'
    }
    stages {
    stage('Checkout') {
      steps {
        sh 'echo passed'
        //git branch: 'main', url: 'https://github.com/mrloki32/food-delivery-app-FE.git'
      }
    }
 stage('Install Dependencies') {
      steps {

        sh 'npm ci'
      }
    }

    stage('Build Project') {
      steps {
        // Build the Angular project
        sh 'npm run build'
      }
    }
         stage('Build and Push Docker Image') {
      environment {
        DOCKER_IMAGE = "mrlokidocs/angular:${IMAGE_TAG}"
        // DOCKERFILE_LOCATION = "/var/lib/jenkins/workspace/demo-cicd/Dockerfile"
        REGISTRY_CREDENTIALS = credentials('docker-cred')
      }
      steps {
        script {
            sh 'cd /var/lib/jenkins/workspace/demo-cicd@tmp/food-delivery-app-FE && docker build -t ${DOCKER_IMAGE} .'
            def dockerImage = docker.image("${DOCKER_IMAGE}")
            docker.withRegistry('https://index.docker.io/v1/', "docker-cred") {
                dockerImage.push()
            }
        }
      }
    } 
        stage('Push to AWS ECR') {
    steps {
        withAWS(credentials: '211125384091') {
            script {
                sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
                sh "docker tag ${DOCKER_IMG}:${IMAGE_TAG} ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPOSITORY}:${IMAGE_TAG}"
                sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPOSITORY}:${IMAGE_TAG}"
            }
        }
    }
}  
        stage('Deploy to AWS ECS') {
            steps {
                script {
                    sh "aws ecs update-service --cluster ${ECS_CLUSTER} --service ${ECS_SERVICE} --force-new-deployment --region ${AWS_REGION}"
                }
            }
        }
    }
    post {
     always {
        emailext attachLog: true,
            subject: "'${currentBuild.result}'",
            body: "Project: ${env.JOB_NAME}<br/>" +
                "Build Number: ${env.BUILD_NUMBER}<br/>" +
                "URL: ${env.BUILD_URL}<br/>",
            to: 'zeroexploit69@gmail.com'                             
        }
    }
}   
