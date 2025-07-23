# Deployment of an edge runtime using docker compose

The following is to create an edge runtime using docker compose.  
All this also works with podman compose, you just need to replace "docker compose" with "podman compose".

## Registry credentials

You need to perform a docker login using your personal registry credentials.  
Set the WHI_CR_SERVER, WHI_CR_USERNAME and WHI_CR_PASSWORD environment variables accordingly before running the following command.  
```
docker login ${WHI_CR_SERVER} --docker-username=${WHI_CR_USERNAME} --docker-password=${WHI_CR_PASSWORD}
```  

## Create a .env file that contains the environment variables

You need to set the following 3 variables in this .env file.
```
SAG_IS_CLOUD_REGISTER_URL=<pairing url>
SAG_IS_EDGE_CLOUD_ALIAS=<edge runtime name>
SAG_IS_CLOUD_REGISTER_TOKEN=<pairing token>
```   

## Start the docker compose stack

The stack is specified in the doccker-compose.yaml file, which you can start using the following command:
```
docker compose up -d
```   

In some cases, you will need to use "docker-compose" instead of "docker compose".  

## Check the deployment

Use the following command to display the container logs:
```
docker logs -f edge
```

## Stop the docker compose stack

The stack is specified in the doccker-compose.yaml file, which you can start using the following command:
```
docker compose down
```   