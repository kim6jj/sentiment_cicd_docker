# sentiment_cicd_docker
CICD Workflow with Docker
- app.py: Main application
- train.py: Script to train and save trained model
- sentiment.tsv: Data file
- requirements.txt: It contains required packages/dependencies
- Dockerfile: To create the docker image
- Template folder: It contains our web page for application
- model folder: It contains our trained model

- First create Dockerfile
- FROM python:3.8-slim-buster #python image with environment pre-installed
- WORKDIR /app        #directory in container to keep code, config, other files
- COPY . /app         #copy files present in local directory image 'app'
- RUN pip3 install -r requirements.txt    
- expost 5000         #port 5000
- CMD ["python", "app.py"]
