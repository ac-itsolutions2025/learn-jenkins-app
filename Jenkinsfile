pipeline {
    agent any

    environment {
        TF_DIR = 'iac'
        AWS_REGION = 'us-east-2'
        TF_VAR_vpc_name = 'acit-vpc'
    }

    stages {
        stage('Setup Terraform') {
            steps {
                sh '''
                    sudo apt-get update -y
                    sudo apt-get install -y unzip wget
                    wget https://releases.hashicorp.com/terraform/1.6.5/terraform_1.6.5_linux_amd64.zip
                    unzip terraform_1.6.5_linux_amd64.zip
                    sudo mv terraform /usr/local/bin/
                    terraform -version
                '''
            }
        }

        stage('Init Terraform') {
            steps {
                sh '''
                    mkdir -p ${TF_DIR}
                    cat > ${TF_DIR}/main.tf <<'EOF'
                    provider "aws" {
                      region = "${AWS_REGION}"
                    }

                    resource "aws_vpc" "acit_vpc" {
                      cidr_block           = "10.100.0.0/16"
                      enable_dns_hostnames = true
                      enable_dns_support   = true
                      tags = {
                        Name = "${TF_VAR_vpc_name}"
                      }
                    }

                    # Public Subnets
                    resource "aws_subnet" "public_1" {
                      vpc_id                  = aws_vpc.acit_vpc.id
                      cidr_block              = "10.100.1.0/24"
                      availability_zone       = "${AWS_REGION}a"
                      map_public_ip_on_launch = true
                      tags = {
                        Name = "${TF_VAR_vpc_name}-public-1"
                      }
                    }

                    resource "aws_subnet" "public_2" {
                      vpc_id                  = aws_vpc.acit_vpc.id
                      cidr_block              = "10.100.2.0/24"
                      availability_zone       = "${AWS_REGION}b"
                      map_public_ip_on_launch = true
                      tags = {
                        Name = "${TF_VAR_vpc_name}-public-2"
                      }
                    }

                    # Private Subnets
                    resource "aws_subnet" "private_1" {
                      vpc_id            = aws_vpc.acit_vpc.id
                      cidr_block        = "10.100.3.0/24"
                      availability_zone = "${AWS_REGION}a"
                      tags = {
                        Name = "${TF_VAR_vpc_name}-private-1"
                      }
                    }

                    resource "aws_subnet" "private_2" {
                      vpc_id            = aws_vpc.acit_vpc.id
                      cidr_block        = "10.100.4.0/24"
                      availability_zone = "${AWS_REGION}b"
                      tags = {
                        Name = "${TF_VAR_vpc_name}-private-2"
                      }
                    }

                    # Internet Gateway
                    resource "aws_internet_gateway" "igw" {
                      vpc_id = aws_vpc.acit_vpc.id
                      tags = {
                        Name = "${TF_VAR_vpc_name}-igw"
                      }
                    }

                    # Public Route Table
                    resource "aws_route_table" "public_rt" {
                      vpc_id = aws_vpc.acit_vpc.id
                      route {
                        cidr_block = "0.0.0.0/0"
                        gateway_id = aws_internet_gateway.igw.id
                      }
                      tags = {
                        Name = "${TF_VAR_vpc_name}-public-rt"
                      }
                    }

                    # Private Route Table
                    resource "aws_route_table" "private_rt" {
                      vpc_id = aws_vpc.acit_vpc.id
                      tags = {
                        Name = "${TF_VAR_vpc_name}-private-rt"
                      }
                    }

                    # Route Table Associations
                    resource "aws_route_table_association" "public_assoc_1" {
                      subnet_id      = aws_subnet.public_1.id
                      route_table_id = aws_route_table.public_rt.id
                    }

                    resource "aws_route_table_association" "public_assoc_2" {
                      subnet_id      = aws_subnet.public_2.id
                      route_table_id = aws_route_table.public_rt.id
                    }

                    resource "aws_route_table_association" "private_assoc_1" {
                      subnet_id      = aws_subnet.private_1.id
                      route_table_id = aws_route_table.private_rt.id
                    }

                    resource "aws_route_table_association" "private_assoc_2" {
                      subnet_id      = aws_subnet.private_2.id
                      route_table_id = aws_route_table.private_rt.id
                    }
                    EOF
                '''
            }
        }

        stage('Terraform Init') {
            steps {
                sh '''
                    cd ${TF_DIR}
                    terraform init
                '''
            }
        }

        stage('Terraform Plan') {
            steps {
                sh '''
                    cd ${TF_DIR}
                    terraform plan -out=tfplan
                '''
            }
        }

        stage('Terraform Apply') {
            steps {
                input message: 'Approve to create VPC?'
                sh '''
                    cd ${TF_DIR}
                    terraform apply -auto-approve tfplan
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Custom VPC 'acit-vpc' created successfully!"
        }
        failure {
            echo "❌ VPC creation failed!"
        }
    }
}
