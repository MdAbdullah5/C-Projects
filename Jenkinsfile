pipeline {
    agent any

    environment {
        GITHUB_REPO = 'https://github.com/reddy756/Bhaskar_InfinitudeIT.git'
        TERRAFORM_DIR = '.'  // Ensure Terraform scripts are in the root directory
        AWS_REGION = 'ap-south-1'  // Mumbai region
    }

    stages {

        // Stage 1: Checkout Code from GitHub
        stage('Checkout Code') {
            steps {
                git url: "${GITHUB_REPO}", branch: 'master'  // Using the 'master' branch
            }
        }

        // Stage 2: Execute Terraform Script to Provision Infrastructure
        stage('Execute Terraform Script') {
            steps {
                dir("${TERRAFORM_DIR}") {
                    sh 'terraform init'
                    sh 'terraform apply -auto-approve'
                }
            }
        }

        // Stage 3: Deployment & Load Balancing for FastAPI App
        stage('Deployment & Load Balancing') {
            steps {
                script {
                    sh '''
                    # SSH into EC2 instance and deploy FastAPI app
                    ssh -i "hello.pem" ec2-user@ec2-65-0-81-193.ap-south-1.compute.amazonaws.com <<EOF
                        # Install necessary packages (if not already installed)
                        sudo yum update -y
                        sudo yum install git -y
                        sudo pip3 install fastapi uvicorn

                        # Clone the GitHub repository
                        git clone ${GITHUB_REPO}

                        # Navigate to the FastAPI app directory
                        cd Bhaskar_InfinitudeIT/app

                        # Run the FastAPI app using uvicorn
                        nohup uvicorn main:app --host 0.0.0.0 --port 8000 &
                    EOF

                    # Configure AWS Elastic Load Balancer
                    aws elb create-load-balancer --load-balancer-name my-load-balancer \
                        --listeners Protocol=HTTP,LoadBalancerPort=80,InstanceProtocol=HTTP,InstancePort=8000 \
                        --availability-zones ${AWS_REGION}a
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Please check the logs for more details.'
        }
        always {
            echo 'Pipeline execution finished.'
        }
    }
}
