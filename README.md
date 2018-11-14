## simple-java-maven-app

This repository is for the
[Build a Java app with Maven](https://jenkins.io/doc/tutorials/build-a-java-app-with-maven/)
tutorial in the [Jenkins User Documentation](https://jenkins.io/doc/).

The repository contains a simple Java application which outputs the string
"Hello world!" and is accompanied by a couple of unit tests to check that the
main application works as expected. The results of these tests are saved to a
JUnit XML report.

## How to Setup GitHub Plugin in Jenkins

1. Install `gitHub Integration` plugin from Manage Jenkins > Manage Plugins
2. Configure github for Jenkins by going to Manage Jenkins > Confgure System
3. Add Github server under ```Github``` section
4. Add credential and choose kind `secret text`
5. Generate access token from your github account and put it in `secret` field in jenkins
6. Click on `Test connection` to test connection 

## How to Setup Jenkins Pipeline

1. Click on `New item` and enter pipeline name
2. Under `Build Triggers` section, tick `GitHub hook trigger for GITScm polling`
3. Under `Pipeline` section, choose `pipeline script from SCM`
4. Choose `git` as SCM
5. Enter the repository URL
6. Add credential
7. Choose `username with password` and enter your github username and password.
If you use 2FA, generate your `access token` at github and enter it in the password field. 
However, if your repository is public, you can even proceed without setting up credential.

## How to Run Jenkins Docker Container in AWS Elastic Container Service (ECS)

1. Create repository in Elastic Container Repository (ECR)
2. Follow the push command in ECR in order to login, build, tag and push docker image to ECR
3. Go to task definitions and create a task. This step is very important because it's where you will setup the run configuration for your docker container based on the image you put in ECR

```
  docker run \
  -u root \
  -p 8000:8080 \
  -v jenkins-data:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v "$HOME":/home \
  jenkinsci/blueocean
  ```
  4. Add voloume under `Volumes` section of task creation
  - Create a volume for mapping the docker container to docker daemon. This mapping is neccessary because it allows `jenkinsci/blueocean` container to communicate with the Docker daemon, which is required if the jenkinsci/blueocean container needs to instantiate other Docker containers. 
  - Create a volume for mapping the `/var/jenkins_home` directory to a Docker volume
  5. Create container based on the docker run command above. 
  - Enter the URI of jenkin-docker image that you pushed in ECR
  - Enter the port `8080:8080` under `Port mappings` section
  - Select `Mount points` - `Source volume` and enter `/var/run/docker.sock` and `/var/jenkins_home` respectively as container path under `Storage and Logging` section. 
  - Enable `Auto-configure CloudWatch Logs` (recommended)
  - Enter `root` in the `User` field under `Security` section
  - Proceed on create container and create task. 
  6. Create cluster (choose EC2 if you want to have SSH capability to ECS/EC2 instance)
  Note: choose the `EC2 instance type` wisely and make sure that its memory is bigger than the memory limit you defined when creating `task`
  7. Create service inside cluster. You will see the `task` you created under `Task Definition` field then proceed as normal to create the service. 
  
For reference, look at [Task definition documentation](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definition_parameters.html#container_definition_storage) to define the task correctly according to docker run command.  

Voila! Enjoy using Jenkins on ECS! 
