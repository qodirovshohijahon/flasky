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
            pip install --upgrade pip
            pip install --upgrade setuptools
            pip install pytest
            pip cache purge
            pip install psycopg2==2.7.3
            pip install -r requirements.txt

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
            pytest
        '''
      }
    }

    stage('Build and Push Docker Image') {
      steps {
        // Build Docker image
        sh 'docker build -t my-flask-app .'
        
        // Configure AWS CLI with credentials for ECR
        withCredentials(
            [
                string(
                    credentialsId: 'aws-credentials', 
                    variable: 'awsCredentials'
                )
            ]
        ) {
          sh '''
            aws configure set aws_access_key_id ${awsCredentials}
            aws configure set aws_secret_access_key ${awsCredentials}
          '''
        }
        
        // Log in to ECR repository
        sh 'aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin $ECR_REPO_URL'
        
        // Tag Docker image
        sh 'docker tag my-flask-app:latest $ECR_REPO_URL:latest'
        
        // Push Docker image to ECR repository
        sh 'docker push $ECR_REPO_URL:latest'
      }
    }
  }
  stage('CD') {
  steps {
    script {
      def sshUser = AWS_ACCESS_KEY_ID // or the name of the IAM user with appropriate permissions
      def sshKey  = credentials('aws-ssh-key') // the name of your Jenkins credentials containing the private SSH key to the EC2 instance
      def sshHost = 'ec2-instance-ip-address' // the public IP address of your EC2 instance
      
      sshagent(['aws-ssh-key']) {
        sh("""
          ssh -o StrictHostKeyChecking=no -i ${sshKey} ${sshUser}@${sshHost} 'docker login -u AWS -p $(aws ecr get-login-password) <aws_account_id>.dkr.ecr.<region>.amazonaws.com'
          ssh -o StrictHostKeyChecking=no -i ${sshKey} ${sshUser}@${sshHost} 'docker pull <aws_account_id>.dkr.ecr.<region>.amazonaws.com/<image_name>'
          ssh -o StrictHostKeyChecking=no -i ${sshKey} ${sshUser}@${sshHost} 'docker stop <container_name>'
          ssh -o StrictHostKeyChecking=no -i ${sshKey} ${sshUser}@${sshHost} 'docker rm <container_name>'
          ssh -o StrictHostKeyChecking=no -i ${sshKey} ${sshUser}@${sshHost} 'docker run -p <host_port>:<container_port> --name <container_name> -d <aws_account_id>.dkr.ecr.<region>.amazonaws.com/<image_name>'
        """)
      }
    }
  }
}
}