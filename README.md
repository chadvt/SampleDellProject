# SampleDellProject
Example of large inventory management solution for retail

>Scenario Summary: I am a warehousing company. On the first and third Friday of every month, our sales spike due to members’ paydays. 
> 
> During these times, traffic from people checking our produce inventory within the stores (and online) goes up significantly so we want to make sure that we are able to accurately display inventory so we are not selling produce that we do not have enough of. 
>
>Anecdotal evidence from cashiers and customers claim inventory lookups are slow during busy peak times. 
>
>The current strategy is to add more resources to the database cluster but I want to assure the business that we are ready to scale at a reasonable cost and inventory is available via browser for members and employees.

# Assumptions:
•	All warehouse stores have network connectivity to database resources, cloud, and local data centers

•	The business will want to provide fault tolerance leveraging on-premise servers and cloud resources as backup

•	Provide 99.99% uptime with <5 second response time for a reasonable cost

 # Tools to Use:

•	 [Docker](https://docs.docker.com/toolbox/toolbox_install_windows/)/[Kubernetes](https://kubernetes.io/) - Containerization will reduce vendor lock-in while allowing for on-prem hardware scaling and cloud resource. Leveraging application based caching solutions (redis for example) while running in multiple regions and local data centers, will ensure rapid response as well as the 99.99% uptime

•	 Elastic Cloud Services ([AWS](https://aws.amazon.com/)) - Containerized approach leveraging a cloud strategy will allow for relevant region specific resources expand and contract as needed while keeping the costs at a minimum. 

# Proposed Workflow:
The workflow I envision is a webapp running onsite at the warehouses as well as customer mobile devices. The workflow would be - Mobile device webapp-->API-->Datastore. Running containerized API apps, you will be able to run in the cloud or on-prem without performance penalties

# Deployment Methodology
Amazon ECS task definitions use Docker images to launch containers on the container instances in your clusters. In this section, you create a Docker image of a simple web application, and test it on your local system or EC2 instance, and then push the image to a container registry (such as Amazon ECR or Docker Hub) so you can use it in an ECS task definition.

# Deployment Steps
Retrieve a Docker Image from a Git Deplository with the necessary function: an Apache webserver that will pull from a (not included) SQL db.

Create a Dockerfile to install our apache server and display output

```FROM ubuntu:12.04```

```# Install dependencies```
```RUN apt-get update -y```
```RUN apt-get install -y apache2```

```# Install apache and display sample webpage```

```COPY index.html/ /var/www/```

```# Configure apache```

```RUN a2enmod rewrite```

```RUN chown -R www-data:www-data /var/www```

```ENV APACHE_RUN_USER www-data```

```ENV APACHE_RUN_GROUP www-data```

```ENV APACHE_LOG_DIR /var/log/apache2```

```EXPOSE 80```

```CMD ["/usr/sbin/apache2", "-D",  "FOREGROUND"]```


Create a container supporting our apache server

```docker build -t dell-test . ``` 

Verify the Container has been created

```docker images --filter reference=dell-test```

Run the image just created

```docker run -p 80:80 dell-test```

Open a browser and point to the server that is running Docker and hosting your container.
If you are running Docker locally, point your browser to http://localhost:8080/.

If you are using docker-machine on a Windows or Mac computer, find the IP address of the VirtualBox VM that is hosting Docker with the docker-machine ip command, substituting machine-name with the name of the docker machine you are using as referenced below.

```docker-machine ip machine-name```
Example - http://192.168.100.99/
You should see a web page.

Stop the Docker container by typing: ```exit```

(Optional) Push your image to Amazon Elastic Container Registry.

Amazon ECR is a managed AWS Docker registry service. Customers can use the familiar Docker CLI to push, pull, and manage images. For Amazon ECR product details, featured customer case studies, and FAQs, see the Amazon Elastic Container Registry product detail pages.

>Note:
>
>This section requires the AWS CLI. If you do not have the AWS CLI installed on your system, see Installing the AWS Command Line >Interface in the AWS Command Line Interface User Guide.
>
>To tag your image and push it to Amazon ECR, create an Amazon ECR repository to store your lamp-stack image. Note the repositoryUri in >the output.

```aws ecr create-repository --repository-name dell-test```

Output:

      {
      
      "repository": {
      
      "registryId": "aws_account_id",
      
      "repositoryName": "lamp-stack",
      
      "repositoryArn": "arn:aws:ecr:us-east-1:aws_account_id:repository/dell-test",
      
      "createdAt": 1505337806.0,
      
      "repositoryUri": "aws_account_id.dkr.ecr.us-east-1.amazonaws.com/dell-test"
      
        }
      
      }

Tag the dell-test image with the repositoryUri value from the previous step.

```docker tag dell-test aws_account_id.dkr.ecr.us-east-1.amazonaws.com/dell-test```

Run the ```aws ecr get-login --no-include-email``` command to get the docker login authentication command string for your registry.

>Note:
>
>The get-login command is available in the AWS CLI starting with version 1.9.15; however, we recommend version 1.11.91 or later for >recent versions of Docker (17.06 or later). You can check your AWS CLI version with the aws --version command. If you are using Docker >version 17.06 or later, include the --no-include-email option after get-login. If you receive an Unknown options: --no-include-email >error, install the latest version of the AWS CLI. For more information, see Installing the AWS Command Line Interface in the AWS >Command Line Interface User Guide.

Copy and paste the results from the previous (get-login) command in the previous step. This command provides an authorization token that is valid for 12 hours.

>Important
>
>When you execute this docker login command, the command string can be visible to other users on your system in a process list (ps -e) >display. Because the docker login command contains authentication credentials, there is a risk that other users on your system could >view them this way and use them to gain push and pull access to your repositories. If you are not on a secure system, you should >consider this risk and log in interactively by omitting the -p password option, and then entering the password when prompted.

Push the image to Amazon ECR with the repositoryUri value from the earlier step.

```docker push aws_account_id.dkr.ecr.us-east-1.amazonaws.com/dell-test```

Next Steps
After the image push is finished, you can use your image in your Amazon ECS task definitions, which you can use to run tasks with.

>Note:
>
>This section requires the AWS CLI. If you do not have the AWS CLI installed on your system, see Installing the AWS Command Line >Interface in the AWS Command Line Interface User Guide.
>
>To register a task definition with the dell-test image

Create a file called dell-test-task-def.json with the following contents, substituting the repositoryUri from the previous section for the image field.

     {
         "family": "dell-test",
         "containerDefinitions": [
           {
            "name": "dell-test",
            "image": "aws_account_id.dkr.ecr.us-east-1.amazonaws.com/dell-test",
            "cpu": 10,
            "memory": 500,
            "portMappings": [
                {
                    "containerPort": 80,
                    "hostPort": 80
                }
            ],
            "entryPoint": [
                "/usr/sbin/apache2",
                "-D",
                "FOREGROUND"
            ],
            "essential": true
        }
    ]
     }
Register a task definition with the dell-test-task-def.json file.

```aws ecs register-task-definition --cli-input-json file://dell-test-task-def.json```

The task definition is registered in the dell-test family as defined in the JSON file.

To run a task with the dell-test task definition

>Important
>
>Before you can run tasks in Amazon ECS, you need to launch container instances into a default cluster. For more information about how >to set up and launch container instances, see Setting Up with Amazon ECS and Getting Started with Amazon ECS using Fargate.
>
Use the following AWS CLI command to run a task with the dell-test task definition.

```aws ecs run-task --task-definition dell-test```


# Deployment Diagram
Please reference the [Proposed Workflow Diagram](https://github.com/chadvt/SampleDellProject/blob/master/WareCostCo.jpg) for a visual reference.
![alt text](https://github.com/chadvt/SampleDellProject/blob/master/WareCostCo.jpg)

# Files in Git
README.md <- This file which overviews the application

dell-test-task-def.json  <- JSON file used in placing the project in AWS

index.html  <- webpage rendered in the local docker container

WareCostCo.jpg   <- Workflow diagram included as reference
