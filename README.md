# Jenkins Pipeline for Java based application using Git, Jenkins, Maven, SonarQube, Docker, Argo CD and Kubernetes
![image](https://github.com/user-attachments/assets/80f14052-d15b-45e9-baf9-0dc5eb0bcad1)

NOTE:
-	We would be using 2 git repositories: 1 for CI part , 1 for CD part
-	We will use declarative jenkisfile bcoz it is easy to use, easily understandable etc.

## Continuous Integration Part:

-	First thing is to get Jenkins pipeline triggered whenever there is a new commit, using webhooks (or >Polling SCM,  >Using Jenkins CLI or API etc). Webhook is the most efficient way because the github webhook only send the trigger to build the Jenkins pipeline only when a webhook is triggered. We can manually set the various options available so as to trigger the webhooks.

-	Then we must create a Jenkinsfile for the pipeline with all required stages. Here we will use docker as an agent which is very light weight and it destroys as soon as the work is done (no resource usage during idle time). Also, with docker agents we don’t have to do lot of installations. Eg: We don’t have to install maven, testing tools or any other packages/sw. We can directly use appropriate image for the container.
Image for the docker agent must have "maven" and "docker" installed. Either create a personal image using dockerfile or use : d2bdocker/jenkins-agent-docker-maven-java:v1

-	For the java app. we will use maven to build the app. in a package along with dependencies. Then, Unit test and static code analysis needs to be done.
  
-	We will also integrate notification tools like stack or email notifications so as to notify the developers for build failures, testing failures, bugs etc.

-	If build passes, we move to next step, that is integrating code scanning tool like SonarQube. Here, we can find any security vulnerability in the app. code. Also, we can check that all criteria are met for the application code files. If there is any issue or security vulnerability then again send notifications. If no then move to next step.

-	In next step docker image of app. files, dependencies, runtime env. Etc will be created with new tag. This image will be created using shell commands in Jenkins file. Also, we will push the image to container registry (any).

## Continuous Delivery Part:

-	Earlier approach to deploy the application image from dockerhub to k8s cluster was to use ansible playbooks or shell scripts. This way the applications were deployed on k8s clusters. But, the main issue with ansible and shell scripts were that they were not scalable and the fact that both methods are not at all designed for continuous delivery with k8s clusters approach.

- Therefore, we had to choose a continuous delivery tool like argo cd, flux cd etc. Gitops approach and its tool like Argod cd relies on a practice, that we must have a separate repository that has only the deployment related yaml files or the “application manifests”. Eg: deploy.yaml, service.yaml, pod.yaml.

-	Whenever new image build happens and pushes to dockerhub. Then, we can update the deployment manifests with the new image name or tag in following ways?
  
     - Method 1: We can use a tool like “Argo Image Updater”. This tool continuously monitors the dockerhub or any other container          registries and whenever new image is pushed to the container registry, then Argo Image Updater will directly update the git          repository of app. manifests with new versions of pod.yaml, deploy.yaml etc. 
       It can also update the helm charts if stored in that repository.

      - Method 2: We can use the jenkinsfile itself and use stages. A. Check into the repository with app. manifests. B. Use shell           commands and update the yaml files with new image versions etc.

      - Method 3: We can create another pipeline for it. 

-	Thus, as soon as there is an update in the repository of app. manifests, the Argo cd tool will create a deployment automatically as per the yaml files present in that repository. The change can be any like changing of image versions, replica counts, volume mount related etc. So, application files are deployed on k8s clusters as per the configurations and thus serving to its customers.


## Spring Boot based Java web application
 
This is a simple Sprint Boot based Java application that can be built using Maven. Sprint Boot dependencies are handled using the pom.xml 
at the root directory of the repository.

This is a MVC architecture based application where controller returns a page with title and message attributes to the view.

## Execute the application locally and access it using your browser

Checkout the repo and move to the directory

```
git clone https://github.com/iam-veeramalla/Jenkins-Zero-To-Hero/java-maven-sonar-argocd-helm-k8s/sprint-boot-app
cd java-maven-sonar-argocd-helm-k8s/sprint-boot-app
```

Execute the Maven targets to generate the artifacts

```
mvn clean package
```

The above maven target stroes the artifacts to the `target` directory. You can either execute the artifact on your local machine
(or) run it as a Docker container.

** Note: To avoid issues with local setup, Java versions and other dependencies, I would recommend the docker way. **


### Execute locally (Java 11 needed) and access the application on http://localhost:8080

```
java -jar target/spring-boot-web.jar
```

### The Docker way

Build the Docker Image

```
docker build -t ultimate-cicd-pipeline:v1 .
```

```
docker run -d -p 8010:8080 -t ultimate-cicd-pipeline:v1
```

Hurray !! Access the application on `http://<ip-address>:8010`


## Next Steps

### Configure a Sonar Server locally

```
apt install unzip
adduser sonarqube
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.4.0.54424.zip
unzip *
chmod -R 755 /home/sonarqube/sonarqube-9.4.0.54424
chown -R sonarqube:sonarqube /home/sonarqube/sonarqube-9.4.0.54424
cd sonarqube-9.4.0.54424/bin/linux-x86-64/
./sonar.sh start
```

Hurray !! Now you can access the `SonarQube Server` on `http://<ip-address>:9000` 


