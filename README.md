# Zomato-Clone-Secure-Deployment-with-DevSecOps-CI-CD

Hello there! This documentation outlines the deployment process for the React JS Zomato clone. Jenkins will guide us through this journey, with Docker serving as our orchestration tool. Delve into the intricacies of these deployment steps to gain insights into the world of DevSecOps within this document.

In this document, we explore the incorporation of security tools at various stages of the DevSecOps process, emphasizing their integral role in enhancing security measures throughout the entire software development lifecycle.
## What is the difference between DevOps and DevSecOps ?
DevOps improves the speed and efficiency of the software development lifecycle to build and deliver software faster and with better quality. DevSecOps focuses on reducing the risk of vulnerabilities in software by integrating security early in the development process

![image](https://github.com/kcheth/Zomato-Clone-DevSecOps/assets/106922418/f67fbe02-21bc-49f9-a1ab-7f01bf247f9c)

### Table of contents
1. Launch an Ubuntu(22.04) t2.large Instance
2. Install Jenkins, Docker and Trivy
   - Install Jenkins
   - Install Docker
   - Install Trivy
4.  Launch a t2.medium instance and install SonarQube Scanner. For installation please refer link
5. Install Plugins like JDK, SonarQube Scanner and NodeJs
   - Configure Java and NodeJs in Global Tool Configuration
   - Configure Sonar Server in Manage Jenkins
7. Create Job
8. Install OWASP Dependency Check Plugins
9. Docker Image Build and Push
10. Terminate instances.
11. Complete Pipeline Script

![image](https://github.com/kcheth/Zomato-Clone-DevSecOps/assets/106922418/b0205c3b-0363-4039-aa4c-d9f488901b8d)

## Step 1 :  Launch an Ubuntu 22.04 t2.large instance
Initiate the launch of an AWS t2.large Instance, utilizing the Ubuntu image. Create a new key pair or leverage an existing one. Ensure the Security Group settings include enabling HTTP and HTTPS, allowing access to all ports (although it's advisable to restrict ports for security purposes, we'll open all ports for learning purposes).

![image](https://github.com/kcheth/Zomato-Clone/assets/106922418/477be3b7-2698-4580-9a20-d4263b1a6d49)

### 2A : Install Jenkins, Docker and Trivy
Connect to your console, and run the below commands to install Jenkins

```
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```
After installing Jenkins, remember to open Inbound Port 8080 in your AWS EC2 Security Group because Jenkins operates on that port. Then, access the Jenkins with your Public IP Address followed by port 80

`<Jenkins-public-ip:8080>`

To unlock Jenkins, run the below command

`sudo cat /var/lib/jenkins/secrets/initialAdminPassword`

![image](https://github.com/kcheth/Zomato-Clone/assets/106922418/340547f5-9a93-41b9-89a2-6b92c5e68e88)

Unlock Jenkins using an administrative password and install the suggested plugins.

![image](https://github.com/kcheth/Zomato-Clone/assets/106922418/d3dd42bf-80c6-4400-a383-206ef0e0378e)

Jenkins will now be setup and install all the libraries. 

![image](https://github.com/kcheth/Zomato-Clone/assets/106922418/e48c756b-6f34-4b14-9227-5a0c86760776)

Enter the above asked credentials and click on Save and Continue.
Jenkins is ready to build to your first pipeline.

![image](https://github.com/kcheth/Zomato-Clone/assets/106922418/8756edaa-607f-4366-bbf1-2e21063ee9f1)

### 2B :  Install Docker 
```
sudo apt-get update
sudo apt-get install docker.io -y
sudo usermod -aG docker $USER   #my case is ubuntu
newgrp docker
sudo chmod 777 /var/run/docker.sock
```
### 2C :  Install Trivy
```
sudo apt-get install wget apt-transport-https gnupg lsb-release -y
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy -y
```
## Step 3 : Setup SonarQube
Launch a t2.medium instance and install SonarQube Scanner. For this document, installing SonarQube using Docker image
`docker run -d --name sonar -p 9000:9000 sonarqube:lts-community`
SonarQube runs on the port 9000, hence expose the same in Security Group of the new instance.

Now SonarQube is up and running. To login, enter username and password. 
`username admin
password admin`
![image](https://github.com/kcheth/Zomato-Clone/assets/106922418/dbf558f6-5ba5-4f5d-ac4a-8c6ce2c37f7c)

Update with New Password. After updating, SonarQube dashboard appears as below

![image](https://github.com/kcheth/Zomato-Clone/assets/106922418/7c191f2d-188b-4b5e-afc0-c8c836bfbabf)

## Step 4 :  Install Plugins
Go to Manage Jenkins →Plugins → Available Plugins →

Install below plugins
1. Eclipse Temurin Installer (Install without restart)
2. SonarQube Scanner (Install without restart)
3. NodeJs Plugin (Install Without restart)

![image](https://github.com/kcheth/Zomato-Clone/assets/106922418/03979c56-dd2b-47bc-ab5f-7d9038c476a8)

### 4A : Configure Java and NodeJS in Global tool configuration

![image](https://github.com/kcheth/Zomato-Clone/assets/106922418/de631960-1ed9-47ea-ab48-c5cd2356e6ab)

Click on Apply and Save

### 4B :  Configure Sonar Server in Manage Jenkins

Go to SonarQube Server, Click on administration → Security → Users → Click on Tokens and Update token → Give a name → Click on Generate token

![image](https://github.com/kcheth/Zomato-Clone/assets/106922418/b47e81a8-8741-4341-9850-484a2ecc35fb)
![image](https://github.com/kcheth/Zomato-Clone/assets/106922418/03e2578a-1a4c-4f12-8dc3-6a42d291f6d0)

Copy the generated token and add it to the Jenkins Credentials. Adding the SonarQube token in Jenkins Credentials is necessary for secure authentication between Jenkins and SonarQube.

![image](https://github.com/kcheth/Zomato-Clone/assets/106922418/0de55c26-1a84-4e6f-a968-3a289b30e843)

The credentials dashboard will be seen as below

![image](https://github.com/kcheth/Zomato-Clone/assets/106922418/4465dafe-1492-4f0a-8625-088d551191ae)

After adding the token, let's update the SonarQube URL in the system so that the scanned results of the code will be displayed in SonarQube. The Configure System option is used in Jenkins to configure various servers. 

![image](https://github.com/kcheth/Zomato-Clone/assets/106922418/3d88fccb-0232-4999-951c-fe8da6a01a5c)

The Global Tool Configuration is employed to set up various tools installed through plugins. In this instance, we will install a Sonar Scanner as part of the configured tools.

![image](https://github.com/kcheth/Zomato-Clone/assets/106922418/e64f5eab-2c0b-4d8a-bb2f-b9d2769707c1)

Add a Quality Gates in SonarQube- Quality Gates in SonarQube serve as a mechanism to set specific criteria for the quality of your code. These criteria are defined based on various metrics, such as code coverage, code duplication, and the number of issues or vulnerabilities.

For this project, Quality Gates are set to default.

![image](https://github.com/kcheth/Zomato-Clone/assets/106922418/57ebca2f-ad5c-42f1-950a-6c24a602872d)

Also configure Webhooks

Go to Administration → Configuration → Webhooks

In SonarQube, a Webhook is a mechanism for the system to notify external services or systems about events that occur within SonarQube. Webhooks are commonly used for integrating SonarQube with other tools, systems, or services to automate processes and streamline workflows.

![image](https://github.com/kcheth/Zomato-Clone/assets/106922418/e05ad8af-0b4e-4a6b-8a89-e5c49b834d1b)

Add details as below 

Name: Jenkins

URL: <http://jenkins-public-ip:8080>/sonarqube-webhook/>

click on create

![image](https://github.com/kcheth/Zomato-Clone/assets/106922418/d57504da-1494-48c8-b4b4-1030db7c8f68)
![image](https://github.com/kcheth/Zomato-Clone/assets/106922418/cb8c9c43-b27d-4e3f-b704-3c1b49a8d9f4)

## Step 5 :  Create a Pipeline Job

Go to Jenkins Dashboard → New Item → Enter an item name → Select ‘Pipeline’ as a Job → click on OK

Optional: Provide a description for the job. In the 'Discard Old Builds' section, select the option and set 'Max # of builds to keep' to 2. This ensures that only the two most recent builds of the pipeline will be retained.

![image](https://github.com/kcheth/Zomato-Clone/assets/106922418/0bc22aee-6f64-4f38-81df-741e388840c7)

Now add a pipeline script as below 

```
pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    
    stages {
        stage ('clean workspace') {
            steps {
                cleanWs()
            }
        }
        stage ('Git Checkout'){
            steps {
                git credentialsId: 'repo-token', url: 'https://github.com/kcheth/Zomato-Clone.git'                
            }
        }
        stage('Gitleaks Scan') {
            steps {
                script {
                    docker.image('zricethezav/gitleaks').inside('--entrypoint=""') {
                       sh "gitleaks detect --report-path gitleaks-report.json"
                   }
                    def gitleaksResult = readFile('gitleaks-report.json')
                    if (gitleaksResult.contains('Leaks found')) {
                        error("Gitleaks detected leaks. Build aborted.")
                    } 
                    else {
                        echo "No leaks found. Continuing with the build."
                    }
                }
            }
        }    
        stage ('SonarQube Analysis'){
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectName=zomato \
                    -Dsonar.projectKey=zomato '''
                }
            }
        }
        stage ('Quality Gate'){
            steps {
                waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
            }
        }
        stage ('Install Dependencies'){
            steps {
                sh "npm install"
            }
        }
```

1. **Clean Workspace**: This stage ensures a fresh workspace by cleaning up any existing files, providing a clean environment for the subsequent tasks.
2. **Git Checkout**: This stage checks out the source code from the specified Git repository, allowing the pipeline to work with the latest version of the code.
3. **Gitleaks Scan**: Utilizing Gitleaks, this stage scans the codebase for potential secrets and sensitive information, halting the pipeline if any leaks are detected.
4. **SonarQube Analysis**: Conducts a SonarQube analysis on the code, providing insights into code quality and potential issues, enhancing overall project visibility.
5. **Quality Gate**: Monitors the SonarQube Quality Gate status, ensuring that the defined quality standards are met before proceeding with subsequent stages.
6. **Install Dependencies**: Installs necessary dependencies, such as Node.js packages, to prepare the environment for building and deploying the application.

The build might fail while performing Gitleaks Scan, please provide the necessary permissions for Docker daemon socket at `unix:///var/run/docker.sock:`

![image](https://github.com/kcheth/Zomato-Clone/assets/106922418/19578fee-61a8-4ffe-9a20-58ebc79294f8)

![image](https://github.com/kcheth/Zomato-Clone/assets/106922418/79c72ecd-53de-4f8d-94d7-03e8326a751d)

As shown below, the initial build failed at the Gitleaks Scan stage due to the aforementioned issue. After granting the required permissions to the docker.socket file, the build successfully completed.

![image](https://github.com/kcheth/Zomato-Clone/assets/106922418/9e0e8a6e-a7fe-41c3-8094-8bc3c8f13b74)

Now see the SonarQube report from its dashboard

![image](https://github.com/kcheth/Zomato-Clone/assets/106922418/fc240e78-b5d2-4034-af0d-84f6493c3aa3)

The generated report indicates a passed status with 1.3k lines. For a more detailed overview, navigate to the 'Issues' section.

## Step 6 : Install OWASP Dependency Check plugins
Go to Dashboard → Manage Jenkins → Plugins → OWASP Dependency-Check. Click on it and install it without restart.

![image](https://github.com/kcheth/Zomato-Clone/assets/106922418/a99f34b1-f219-47e1-aed4-42abf12c850b)

Now that Plugin is configured, next is to configure the tool Go to Dashboard → Manage Jenkins → Tools 

![image](https://github.com/kcheth/Zomato-Clone/assets/106922418/b54fb4f9-7e93-404f-ba65-8b01338e9843)

Click on Apply and Save.

Proceed to configure the pipeline by incorporating the following stage into the pipeline, then initiate the build.
```
        stage('OWASP FS SCAN') {
            steps {
                 dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-check'
                 dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
```
The 'OWASP FS SCAN' stage utilizes Dependency-Check to perform a security scan on the project's dependencies, disabling Yarn and Node audits. The results are published using Dependency-Check Publisher.

The 'TRIVY FS SCAN' stage employs Trivy to conduct a filesystem scan on the project, generating a report in the 'trivyfs.txt' file for further analysis.

The build is successful and the Stage View will look as below.

![image](https://github.com/kcheth/Zomato-Clone/assets/106922418/cd3c7fcd-25b2-4f64-a8a4-4a18a3bf5ac0)

Also we can see the Dependency-Check Results

![image](https://github.com/kcheth/Zomato-Clone/assets/106922418/df151979-0375-420b-ba60-0c7e7f8fc721)

## Step 7 : Docker Image Build and Push

Install the below plugins to install Docker tool in our system 

```
Docker

Docker Commons

Docker Pipeline

Docker API

docker-build-step
```
Once the above plugins are installed, configure the Docker in Global Tool Configuration

![image](https://github.com/kcheth/Zomato-Clone/assets/106922418/09ae4f0a-b33b-4832-a60d-97fbac4b3f76)

Also update the DockerHub credentials under Global Credentials 

![image](https://github.com/kcheth/Zomato-Clone/assets/106922418/c22977ee-09d5-4631-bbe0-49e1b85b2f22)

Add the below Build and Push stage to the script

```
    stage('Docker Build & Push') {
           steps {
                cleanWs()
                  script {
                            git credentialsId: 'repo-token', url: 'https://github.com/kcheth/Zomato-Clone.git'
                            withCredentials([usernamePassword(credentialsId: 'docker', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                            sh "echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin"
                              }
                                sh "docker build -t kcheth/zomato:v1 ."
                                sh "docker push kcheth/zomato:v1"
                        }
                }
        }
        stage ('TRIVY'){
            steps {
                sh "trivy image kcheth/zomato:v1 > trivy.txt"
            }
        }
```
You will see the output as below, with the Dependency-check Trend

![image](https://github.com/kcheth/Zomato-Clone/assets/106922418/d518d6af-976c-41b9-a0d4-eb4610776d5a)

After logging in to DockerHub, you'll notice the creation of a new image kcheth/zomato:v1

![image](https://github.com/kcheth/Zomato-Clone/assets/106922418/369fd7ea-0af7-40d8-86ea-83f4c345fc4f)

**Deployment**: Verify the application's functionality by running the container using the following stage.

```
      stage ('Deploy to Container'){
            steps {
                sh 'docker run -d --name zomato -p 3000:3000 kcheth/zomato:latest'
            }
        }
```
Stage View- As shown below, all the stages are successfully passed. 

![image](https://github.com/kcheth/Zomato-Clone/assets/106922418/d024d081-2ff2-40cd-8903-dd423d210cf6)

The application can be accessed at `<Jenkins-public-ip:3000>`

![image](https://github.com/kcheth/Zomato-Clone/assets/106922418/49a55eed-1e15-4dc1-bf96-a81737caca78)

## Step 8 : Clean the resources

Optimize resource usage by terminating AWS EC2 Instances for cost-effectiveness and environmental responsibility. Utilize AWS management tools or commands to gracefully shut down and terminate the Ubuntu(22.04) t2.large and t2.medium Instances. This concludes the deployment process while ensuring operational efficiency.

### Complete Pipeline Script

```
pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    
    stages {
        stage ('clean workspace') {
            steps {
                cleanWs()
            }
        }
        stage ('Git Checkout'){
            steps {
                git credentialsId: 'repo-token', url: 'https://github.com/kcheth/Zomato-Clone.git'
                
            }
        }
        stage('Gitleaks Scan') {
            steps {
                script {
                    docker.image('zricethezav/gitleaks').inside('--entrypoint=""') {
                       sh "gitleaks detect --report-path gitleaks-report.json"
                   }
                    def gitleaksResult = readFile('gitleaks-report.json')
                    if (gitleaksResult.contains('Leaks found')) {
                        error("Gitleaks detected leaks. Build aborted.")
                    } 
                    else {
                        echo "No leaks found. Continuing with the build."
                    }
                }
            }
        }    
        stage ('SonarQube Analysis'){
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectName=zomato \
                    -Dsonar.projectKey=zomato '''
                }
            }
        }
        stage ('Quality Gate'){
            steps {
                waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
            }
        }
        stage ('Install Dependencies'){
            steps {
                sh "npm install"
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                 dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-check'
                 dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage('Docker Build & Push') {
           steps {
                cleanWs()
                  script {
                            git credentialsId: 'repo-token', url: 'https://github.com/kcheth/Zomato-Clone.git'
                            withCredentials([usernamePassword(credentialsId: 'docker', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                            sh "echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin"
                              }
                                sh "docker build -t kcheth/zomato:v1 ."
                                sh "docker push kcheth/zomato:v1"
                        }
                }
        }
        stage ('TRIVY'){
            steps {
                sh "trivy image kcheth/zomato:v1 > trivy.txt"
            }
        }
        stage ('Deploy to Container'){
            steps {
                sh 'docker run -d --name zomato -p 3000:3000 kcheth/zomato:v1'
            }
        }
    }
}
```
### Conclusion: 
In summary, the successful implementation of a DevSecOps CI/CD pipeline for the Zomato Clone Project involved the utilization of key tools and plugins, including GitHub, Jenkins, SonarQube, NodeJs, OWASP Dependency Check, Trivy, Gitleaks, and Docker. The pipeline incorporated comprehensive scans for dependency checks, security, and vulnerabilities at each stage, highlighting the significance of a security-focused approach in the Software Development Life Cycle (SDLC). This project provided valuable insights into the integration of security measures to enhance the overall SDLC process.























































