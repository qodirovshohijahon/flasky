pipeline {

  agent any
    environment {
        AWS_ACCESS_KEY_ID     = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
    }

  stages {
    stage('Checkout') {
      steps {
          checkout(
            [
                $class: 'GitSCM',
                branches: [[name: ' * /master']], 
                extensions: [], 
                userRemoteConfigs: [[url: 'https://github.com/qodirovshohijahon/flasky.git']]
            ]
        )
      }
    }
    stage('Activating and Installing Tools') {
      steps {
        // Set up Python virtual environment
        sh '''
            python3 -m venv venv
            . venv/bin/activate
            pip install pytest
        '''
        // sh '. venv/bin/activate #source venv/bin/activate'
        // Install dependencies
        // sh 'pip install -r requirements.txt'
        // Run tests
        // sh 'pytest'
      }
    }

    stage('Installing dependencies and Running Tests') {
      steps {
        sh '''
            pip install -r requirements.txt
            pytest
        '''
      }
    }


    // stage('Build and Push Docker Image') {
    //   steps {
    //     // Build Docker image
    //     sh 'docker build -t my-flask-app .'
        
    //     // Configure AWS CLI with credentials for ECR
    //     withCredentials(
    //         [
    //             string(
    //                 credentialsId: 'aws-credentials', 
    //                 variable: 'awsCredentials'
    //             )
    //         ]
    //     ) {
    //       sh '''
    //         aws configure set aws_access_key_id ${awsCredentials}
    //         aws configure set aws_secret_access_key ${awsCredentials}
    //       '''
    //     }
        
    //     // Log in to ECR repository
    //     sh 'aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin $ECR_REPO_URL'
        
    //     // Tag Docker image
    //     sh 'docker tag my-flask-app:latest $ECR_REPO_URL:latest'
        
    //     // Push Docker image to ECR repository
    //     sh 'docker push $ECR_REPO_URL:latest'
    //   }
    // }
  }
}