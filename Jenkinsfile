#!/usr/bin/env groovy

library identifier: 'jenkins-shared-library@master', retriever: modernSCM(
  [$class: 'GitSCMSource',
  remote: 'https://gitlab.com/twn-devops-bootcamp/latest/12-terraform/jenkins-shared-library.git',
  credentialsId: 'gitlab-credentials'
  ]
)

pipeline {   
  agent any
  tools {
    maven 'Maven'
  }
  environment {
    IMAGE_NAME = 'prashzan/demo-app:java-maven-2.0'
  }
  stages {
    stage("build app") {
      steps {
        script {
          echo 'building application jar...'
          buildJar()
        }
      }
    }
    stage("build image") {
      steps {
        script {
          echo 'building docker image...'
          buildImage(env.IMAGE_NAME)
          dockerLogin()
          dockerPush(env.IMAGE_NAME)
        }
      }
    }


     stage("provision server") {
      environment {
        AWS_ACCESS_KEY_ID = credentials('jenkins_aws_access_key_id')
        AWS_SECRET_ACCESS_KEY = credentials('jenkins-aws_secret_access_key')
        // how do we overide, or set a value of a variable inside terraform configuration from jenkinsfile, one of ways to do that is 
        // using terraform env variables, using TF_VAR prefix
        TF_VAR_env_prefix = 'test' 
      }
      steps {
        script {
          dir('terraform') {
            sh "terraform init"
            sh "terraform apply --auto-approve"
            /*we need to save result of terraform output ec2-public_ip command into env varibale name EC2_PUBLIC_IP
              inorder to access later on next stage.
              so EC2_PUBLIC_IP variable is accessible on other stage.
            */
            EC2_PUBLIC_IP = sh(
              script: "terraform output ec2-public_ip",
              returnStdout: true
            ).trim()
          }
        }
      }
    }

 

    stage("deploy") {
      /* we have deploy stage which is gonna connect to the instance via ssh,and run scp command to copy our server cmds and docker
      compose file to the ec2 instance before sshing into the server
      */ 
      steps {
        script {
          echo "waiting for EC2 server to initialize"
          /* after ec2 instance is created, it needs some time to initialize, while in the process, we have to wait on this stage 
           this stage commands needs to run after terraform provision the ec2 instance in the above stage.
           */
          sleep(time: 90, unit: "SECONDS")

          echo 'deploying docker image to EC2...'
          echo "${EC2_PUBLIC_IP}"
          
          def shellCmd = "bash ./server-cmds.sh ${IMAGE_NAME}"
          def ec2Instance = "ec2-user@${EC2_PUBLIC_IP}"

          sshagent(['server-ssh-key']) {
            sh "scp -o StrictHostKeyChecking=no server-cmds.sh ${ec2Instance}:/home/ec2-user"
            sh "scp -o StrictHostKeyChecking=no docker-compose.yaml ${ec2Instance}:/home/ec2-user"
            sh "ssh -o StrictHostKeyChecking=no ${ec2Instance} ${shellCmd}"
          }
        }
      }

    }               
  }
}
