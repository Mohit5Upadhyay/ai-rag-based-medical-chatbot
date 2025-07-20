## How to Deploy GenAI Project ? 


# AWS-CICD-Deployment-with-Github-Actions

## 1. Login to AWS console.

## 2. Create IAM user for deployment with only specific policy permissions > from User > create new user.

	#with specific access

	1. EC2 access : It is virtual machine

	2. ECR: Elastic Container registry to save your docker image in aws (like docker hub for registry image)


	#Description: About the deployment

	1. Build docker image of the source code

	2. Push your docker image to ECR

	3. Launch Your EC2 

	4. Pull Your image from ECR in EC2

	5. Lauch your docker image in EC2

	#Policy:

	1. AmazonEC2ContainerRegistryFullAccess

	2. AmazonEC2FullAccess

    #Click security credentials to create Access & Secret Key for AWS Auth.

	
## 3. Create ECR repo to store/save docker image
    - create REPO  with name and other thing default
    - create and Copy URI
    - Save the URI: 315865595366.dkr.ecr.us-east-1.amazonaws.com/medicalbot

	
## 4. Create EC2 machine (Ubuntu-Deployment) - default setting for Free Tier Account:

## 5. Open EC2 and Install docker in EC2 Machine:
	
	
	#optinal

	sudo apt-get update -y

	sudo apt-get upgrade
	
	#required steps:

	curl -fsSL https://get.docker.com -o get-docker.sh

	sudo sh get-docker.sh

	sudo usermod -aG docker ubuntu

	newgrp docker

    docker --version # shows version 
	
# 6. Configure EC2 as self-hosted runner:

    1. 

    Gihtub Setting > Actions > Runner> New self hosted runner > Choose os Linux> then run command one by one into AWS Console to connect with AWS for CI/CD

    after configure commands ->  give the name : self-hosted (Name Runner)


# 7. Setup github secrets:

Add Project Secrets and Variables :

Setting Github Project >  Secrets & Variable > Actions > Repository Secrets > New secrets (ADD all the secrets one by one)

   - AWS_ACCESS_KEY_ID
   - AWS_SECRET_ACCESS_KEY
   - AWS_DEFAULT_REGION
   - ECR_REPO
   - PINECONE_API_KEY
   - OPENAI_API_KEY

# 8. Create Dockerfile 
```docker
FROM python:3.10-slim-buster

WORKDIR /app

COPY . /app

RUN pip install -r requirements.txt

CMD ["python3", "app.py"]
```



# 8. Create .github/workflows/cicd.yml Folder  (work flow)

```yml
name: Deploy Application Docker Image to EC2 instance

on:
  push:
    branches: [main]


jobs:
  Continuous-Integration:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ secrets.AWS_DEFAULT_REGION }}

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v1

      - name: Build, tag, and push image to Amazon ECR
        id: build-image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          ECR_REPOSITORY: ${{ secrets.ECR_REPO }}
          IMAGE_TAG: latest
        run: |
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .  
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
          echo "::set-output name=image::$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG"

  Continuous-Deployment:
    needs: Continuous-Integration
    runs-on: self-hosted
    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ secrets.AWS_DEFAULT_REGION }}

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v1

      - name: Run Docker Image to serve users
        run: |
         docker run -d -e AWS_ACCESS_KEY_ID="${{ secrets.AWS_ACCESS_KEY_ID }}" -e AWS_SECRET_ACCESS_KEY="${{ secrets.AWS_SECRET_ACCESS_KEY }}" -e AWS_DEFAULT_REGION="${{ secrets.AWS_DEFAULT_REGION }}" -e PINECONE_API_KEY="${{ secrets.PINECONE_API_KEY }}" -e OPENAI_API_KEY="${{ secrets.OPENAI_API_KEY }}" -p 8080:8080 "${{ steps.login-ecr.outputs.registry }}"/"${{ secrets.ECR_REPO }}":latest
```

# 9. When any changes occur it will detect automatically and automatically deploy project.

# 10. Edit Inbound Rules for EC2 to access and PORT Mapping
```
custom TCP > 8080 > 00000/0


# Access the WEBSITE: PUBLIC-IP-EC2:8080
```
