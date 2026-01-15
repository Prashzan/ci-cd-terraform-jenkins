### CI/CD pipeline using jenkins using terraform as a infrastructure provisioning

### Terraform will provision the create a new server for us so that we can deploy our app.. which leds us automate the part of creating remote server using ci/cd pipelines. in order to do this


we have to do some things:

#### 1. we gonna need to create a key pair for the server.
#### 2. Install terraform inside jenkins container, cuz we want to execute terraform commands inside jenkins pipeline
#### 3. We are going to create terraform configuration files inside the project, so that we can do terraform apply inside the folder, where we have all things. we gonna have terraform configuratin file part of our application code. (best practice to include everything that our app needs, including infra automation, app configuration automation inside the application code itself, so that ci/cd pipeline has access to them

#### 4. Adjust jenkinsfile

#### To create a key pair

We can either go inside jenkins container and create public private key using ssh-keygen or as an alternative we can create a key pair inside aws manually and create a credentials in jenkins from the key pair. we are gonna do the 2nd one, creating key pair in aws manually. 

#### when we are pulling image from private repo, we first have to do dockerlogin so that the server where we are trying to pull that image to authenticates with private repository

#### To run the project
#### -- First set docker hub login credentials in jenkins credentials UI
#### -- Then set aws access key and secret access key in jenkins credentials UI
#### -- create a ssh with private key credential in jenkins credentials UI with username ec2-user and content for private key is the content of the .pem file that aws give us while creating key-value pair. this is used to ssh into the ec2-instance from jenkins server.
#### -- Run the pipeline. 

#### To test
#### -- just ssh into the server using the ip that terraform provides while provisioning the server.

------------------------------------ Remote state in Terraform using S3 bucket ---------------------------------

#### while executing the pipeline through jenkins, it created terraform state file inside jenkins file, we don't have it locally. So how do we share the terraform state between different environment and different team members.

#### There is a simple way to do that is to configure a remote terraform state, basically a remote storage where terraform state file will be stored. it is good for data back up incase something happens to the server.

#### To access the bucker locally
```
terraform init
```

```
terraform state list
```


