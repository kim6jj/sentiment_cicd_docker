# sentiment_cicd_docker
CICD Workflow with Docker (Source: Deepak Moonat)
- app.py: Main application
- train.py: Script to train and save trained model
- sentiment.tsv: Data file
- requirements.txt: It contains required packages/dependencies
- Dockerfile: To create the docker image
- Template folder: It contains our web page for application
- model folder: It contains our trained model

#First create Dockerfile
- FROM python:3.8-slim-buster #python image with environment pre-installed
- WORKDIR /app        #directory in container to keep code, config, other files
- COPY . /app         #copy files present in local directory image 'app'
- RUN pip3 install -r requirements.txt    
- expost 5000         #port 5000
- CMD ["python", "app.py"]

#create docker image using dockerfile
- while local, 'docker build -t app:v1 . (confirm by by issuing docker images)
- can also test locally running docker run -p 5000:5000 app:v1 (http://localhost:5000 to see html app)

#push to dockerhub
- docker tag app:v1 kim6jj/app:v1.0
- docker push kim6jj/app:v1.0 (logged into dockerhub using docker for windows desktop)
- anyone can now pull from my docker the container app (docker pull kim6jj/app:v1.0)


#run and deploy containerized app on AWS fargate (serverless service)
- go to elastic container service ECR and create a repo /sentiment_analysis
- using docker image created before, we can push using AWS CLI 
- configure AWS account credentials to execute AWS commands (grab access key and secret from security credentials)
- run 'aws configure'
      - aws ecr get-login-password --region us-west-1 | docker login --username AWS --password stdin xxxxxxxxxxxxxxxxx.com
- docker tag app:v1   xxxxxxxxxxxxxxxxxxx/sentiment_analysis:latest (repo uri)
- docker push xxxxxxxxxxxxxxxxxxx/sentiment_analysis:latest (push to repo)

#create cluster
- within ECR, we now create a cluster with 'Networking only' (to create task definition within cluster to reach via public IP)
- now create task definition under 'view cluster' and select Fargate as launch type
- configure with task def name, memory and cpu (1GB, .25 vCPU)
      - add container we pushed to ECR repo via URI provided within image pushed to ECR repo, also add port mappings '5000 TCP'
- after creation, select and run task under actions selecting the cluster VPC and subnet
      - task is now created successfully and we now need to add an inbound rule to sec group to access our application on port 5000 (sec group is stateful so inbound changes automatically reflect to outbound as well)
- under ENI Id under network section will take you to network interface where security groups is available
      - under sec group, edit inbound rule and add a custom TCP rule with port 5000 and source to be all (0.0.0.0/0) and save
      - we can now access our container app via Public IP under network section of task defintion adding port 5000 (x.x.x.x:5000)

#CI/CD Pipeline using AWS CodeCommit, CodeBuild, Pipeline and ECS + Fargate from earlier
- Create an App Load Balancer ALB in EC2 (to make app scalable + HA)
- using the task defintition from before, use the same task def to create a Fargate service (auto scaling grp for tasks, defines number of tasks to run across cluster, where they should be running, automatically associates them with a load balancer, and horiztontally scales based one metrics like mem util, etc)
- Fargate using the same cluster from before - also using the same cluster VPC and appropriate subnets (ex. us-west-1a, us-west-1b)
- using the ALB we created and add rule in security group to allow TCP 5000
      - can verify app is running by visiting the DNS name of the ALB in a browser

#Now that the service is running, can build a CI/CD Pipeline
- AWS CodeCommit is a version control similar to GitHub 
- create a repo and using IAM attach 'AWSCodeCommitPowerUser' policy to user
      - also generate HTTPS Git Credentials for AWS CodeCommit and generate
- 
