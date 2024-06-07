Setting up the infrastructure for Continuous Integration:
1. jenkins server (port 8080)
2. sonarqube server (port 9000)
3. nexus server (port 8081)
4. slave machine

   
Jenkins server: A Jenkins server is an automation tool used for continuous integration (CI) and continuous delivery (CD) in software development. It helps automate the process of building, testing, and deploying code changes, ensuring that code quality is maintained and deployment is consistent. Jenkins integrates with various tools and platforms through plugins, supports defining workflows as code, and can scale across multiple machines to handle large workloads. It is widely used to improve development efficiency and ensure early detection of issues.
setup:
1. Install the required version of java as jenkins is built in java and run on Java Virtual Machine (JVM)
    1. sudo apt update
    2. sudo apt -get install openjdk-17-jdk
   
    ![image](https://github.com/venkatesh-reddy679/Board_Game-CI-CD/assets/60383183/c01b7211-ebb6-47fb-8871-4a9954220068)
   
2. Install jenkins
    1. sudo apt install apt-transport-https -y
    2. wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo apt-key add -
    3. sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
    4. sudo apt update
    5. sudo apt install jenkins -y
    6. sudo systemctl start jenkins
    7. sudo systemctl enable jenkins
    and the jenkins server is exposed on port 8080 by default. Make sure to keep the port open on the virtual machine and on firewall if any
   
   ![image](https://github.com/venkatesh-reddy679/Board_Game-CI-CD/assets/60383183/685691ab-91d0-4b55-9644-a64d40d6cab1)
   
   retrieve the password from the given file, proceed with installation of required plugins, and create first admin user.
   
   ![image](https://github.com/venkatesh-reddy679/Board_Game-CI-CD/assets/60383183/63a27b55-b43a-4414-80e2-01189d9d880f)

3. Install Docker on the same jenkins server to run sonarqube and nexus repository as docker containers.
    1. sudo apt-get update
    2. sudo apt-get install ca-certificates curl
    3. sudo install -m 0755 -d /etc/apt/keyrings
    4. sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
    5. sudo chmod a+r /etc/apt/keyrings/docker.asc
    6. echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    7. sudo apt-get update
    8. sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
       
    ![image](https://github.com/venkatesh-reddy679/Board_Game-CI-CD/assets/60383183/3b500ba5-53ad-458b-bb86-69afb39809cc)

    to enable any user like jenkins user to run the docker commands as the root user, like creating containers, deleting containers, etc.
   
    ![image](https://github.com/venkatesh-reddy679/Board_Game-CI-CD/assets/60383183/869cae3c-fadf-4474-842c-9336eaff5dff)

Install the below mentioned plugins . go to manage jenkins -> plugins -> available plugins
1. eclipse temurian installer (for installing and using the required version of jdk on agent)
2. config file provider (required to create a maven setting.xml file for deploying the build artifact to the nexus repository)
3. sonarqube-scanner (tool to perform automatic code review)
4. docker
5. docker pipeline
6. kubernetes
7. kubentetes CLI
8. pipeline maven intgration
9. maven integration.

To use the third-party tools installed as plugins in the jenkins pipelines, Configure the installed plugins as tools in manage jenkins -> tools
![image](https://github.com/venkatesh-reddy679/Board_Game-CI-CD/assets/60383183/c79dfd71-ef86-4a5e-baea-ec333f80c62d)
![image](https://github.com/venkatesh-reddy679/Board_Game-CI-CD/assets/60383183/5e50bcdd-d374-43f1-a251-2eecd60e32cc)
![image](https://github.com/venkatesh-reddy679/Board_Game-CI-CD/assets/60383183/4b1e0c71-063f-4a50-ad76-5de8442f4eac)
![image](https://github.com/venkatesh-reddy679/Board_Game-CI-CD/assets/60383183/d285e870-40c7-48bc-8730-c223e7b7a71b)



SonarQube:  SonarQube is an open-source platform used for continuous inspection of code quality through static code analysis, detecting bugs, code smells, and security vulnerabilities across more than 25 programming languages. It provides detailed metrics and reports, integrates seamlessly with CI/CD tools like Jenkins, and allows customization of coding rules to enforce standards. By using SonarQube, development teams can improve code quality, maintainability, and security, receive continuous feedback, and ensure compliance with coding best practices, making it an essential tool for maintaining high standards in software development projects.

Sonarqube-scanner plugin performs the code quality analysis, generates the reports, and publish the reports to the configured sonarqube server.

setup: (running the sonarqube server on the same jenkins server as a docker container and publishing the service on port 9000 of the host)
![image](https://github.com/venkatesh-reddy679/Board_Game-CI-CD/assets/60383183/14e01a58-5c9d-4b53-9dcc-9a15f61d6e68)
![image](https://github.com/venkatesh-reddy679/Board_Game-CI-CD/assets/60383183/efbbf27c-0e65-4951-900f-b7a78574644a)
default username: admin && default password: admin

steps to configure the sonarqube server in Jenkins:

1. create a security token in the sonarqube server. go to -> administration -> security -> users (choose the user  you want to authenticatse as) -> click on tokens & generate a new token and save it somewhere
![image](https://github.com/venkatesh-reddy679/Board_Game-CI-CD/assets/60383183/7b913052-6394-4c0e-9260-bf3aba162c10)

2. store the generated security token in the global credentials as a "secret text" in Jenkins server. go to manage jenkins -> credentials -> click on global & click on add credentials
![image](https://github.com/venkatesh-reddy679/Board_Game-CI-CD/assets/60383183/ac65c3e5-bfce-4a00-916f-84391c55b2d3)

3. configure the sonarqube server in jenkins server. go to manage jenkins -> system and scroll to sonarqube server settings.
![image](https://github.com/venkatesh-reddy679/Board_Game-CI-CD/assets/60383183/18453c27-699a-4f54-a388-16b5421b7082)



Nexus repository: Nexus Repository is a repository manager that stores, manages, and distributes software artifacts required for development, deployment, and provisioning. It supports various artifact formats like Maven, npm, Docker, and more, and can proxy external repositories like maven central, docker hub to cache artifacts locally, improving build performance and reliability. 

setup: (running the nexus repository as docker container on the same jenkins server)
![image](https://github.com/venkatesh-reddy679/Board_Game-CI-CD/assets/60383183/2fa47874-3080-4559-90f2-84d88a043b0b)
![image](https://github.com/venkatesh-reddy679/Board_Game-CI-CD/assets/60383183/a689df50-0ca5-4c25-b3d4-7e0c2978f2f1)

steps to configure the nexus repository for to use in te jenkins pipeline:
1. update the maven project pom.xml file with "distributionmanagement" tag and provide the maven-releases and maven-snapshots url's
   ![image](https://github.com/venkatesh-reddy679/Board_Game-CI-CD/assets/60383183/71fae80b-482d-4e11-8027-c1edb777a60a)
   ![image](https://github.com/venkatesh-reddy679/Board_Game-CI-CD/assets/60383183/f1849c52-522f-450a-a10e-5ce1710cd939)
2. create a maven setting.xml file to provide the credentials for these repositories (this is why we installed config file provider plugin).
    go to manage jenkins -> managed files -> Add a new config -> select maven setting.xml (provide a valid name) -> in content section, scroll to servers and update the credentials accordingly. (credentials are same as nexus)
   ![image](https://github.com/venkatesh-reddy679/Board_Game-CI-CD/assets/60383183/19038569-870e-4c22-904c-4196b5c79dc7)



Slave machine:
1. create a virtual machine, install java version same as on jenkins server, install docker and run "chmod 666 /var/run/docker.sock" as root user that enables any user to run docker commands.
   Generate ssh key pair using "ssh-keygen -t rsa" command, copy the public key and store it in .ssh/authorized_keys file.
   ![image](https://github.com/venkatesh-reddy679/Board_Game-CI-CD/assets/60383183/3d4eeafc-a42d-496c-ac3f-1adb4fa2de9e)

2. copy the private key and store it in the jenkins server as a global credentials as "SSH username with private key"
    ![image](https://github.com/venkatesh-reddy679/Board_Game-CI-CD/assets/60383183/4783be58-320a-43d4-8f64-958ee654b62a)
3. go to manage jenkins -> nodes -> add new node
   ![image](https://github.com/venkatesh-reddy679/Board_Game-CI-CD/assets/60383183/12ee09f1-c02b-4f8c-961f-cd2898b4a66a)
   ![image](https://github.com/venkatesh-reddy679/Board_Game-CI-CD/assets/60383183/f80c3934-032e-4b31-afb6-e0b94bab2aa0)



Installing Trivy on slave machine:

Trivy is an open-source vulnerability scanner used to detect the known vulnerabilities in various components of software development such as docker images, file systems, git repositories and kubernetes clusers. It help us ensure that software is developed deployed with no existing issues.

Jenkins doesn't provide any default plugin for trivy, so we have to manually install it on agent machine.

commands to install trivy:
1. sudo apt-get install wget apt-transport-https gnupg lsb-release
2. wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
3. echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
4. sudo apt-get update
5. sudo apt-get install -y trivy

![image](https://github.com/venkatesh-reddy679/Board_Game-CI-CD/assets/60383183/3fa2a8d8-ef9a-4f9e-b01e-85f366867b6c)


Setting up the jenkins pipeline for continuous Integration:

1. click on new item and select pipeline as type of jenkins job

![image](https://github.com/venkatesh-reddy679/Board_Game-CI-CD/assets/60383183/3766cfa2-3343-4e82-b35c-c29a7fc7b6d0)

2. setting the agent to run the pipeline, and use the tools block to define the tools that we installed as third-party plugins to use in this pipeline

![image](https://github.com/venkatesh-reddy679/Board_Game-CI-CD/assets/60383183/2eda3ece-f883-4440-b52f-92bc9ba9c871)


3. Stages for cloning the git repository, compiling the source code, testing the compiled source code, and packaging the project to a jar file

![image](https://github.com/venkatesh-reddy679/Board_Game-CI-CD/assets/60383183/7ca40a7b-9094-4b48-ab04-cc7d3fe2f77f)

   make sure to update the version tag in pom.xml to get dynamic version for the build artifact using the current build number

![image](https://github.com/venkatesh-reddy679/Board_Game-CI-CD/assets/60383183/ca697bd1-7617-46c1-83d7-5b4bc1698d30)


4. Stage to scan the directory after cloning the git repository for any known vulenrabilities using trivy that writes the output in table format to a file specified in -o flag and . represents current directory

![image](https://github.com/venkatesh-reddy679/Board_Game-CI-CD/assets/60383183/c03a3164-6d2d-4c11-918a-8706a5cdcfac)


5. Stage for code quality check and quality gate
   
withSonarQubeEnv('sonarqube-server') block sets up the environemnt for sonarqube code quality analysis where  "sonarqube-server" is the name with which the sonarqube server is configured in jenkins. The maven command "mvn sonar:sonar" triggers the SonarQube analysis for a Maven-based project, sending the analysis data to a SonarQube server.

A Quality Gate in sonarqube server is a set of conditions that a project must meet to be considered of acceptable quality. It is a way to enforce a minimum standard of quality before changes are integrated into the main codebase. These conditions can include metrics like code coverage, number of bugs, code smells, duplications, and other issues.

![image](https://github.com/venkatesh-reddy679/Board_Game-CI-CD/assets/60383183/5e4c8999-ad0f-45fb-bf38-8df5dc5d4a61)

"sonar way" is the default quality gate our project points to. we can a new quality gate with different coverage level by clicking on create button. To set our project to use a customized quality gate, got to project -> project settings -> quality gate -> always use a specific quality gate -> choose the gate

![image](https://github.com/venkatesh-reddy679/Board_Game-CI-CD/assets/60383183/6556d90a-e0a9-46d8-86e0-bf17c6e86b5f)

"waitForQualityGate" step waits until the sonarqube analysis is completed and sonarqube server send the quality gate status to jenkins server  using the webhook that we set on sonarqube server and "abortPipeline: true" will abort the pipeline if the quality gate is failed.

It is a good practice to wrap "waitForQualityGate" in a "timeout" block to prevent the build from waiting indefinitely in case of issues with SonarQube analysis. The timeout block will limit the maximum time the build waits for the Quality Gate status. If the timeout expires before the quality gate status is received, the pipeline will throw a timeout error. If the quality gate status is received before the timeout expired, waitForQualityGate will be passed and pipeline execution will be resumed.

![image](https://github.com/venkatesh-reddy679/Board_Game-CI-CD/assets/60383183/e1f46ae7-affe-4248-ae8d-cfe80c1ea0cd)

6. Stage to deploy the generated build atrifact to nexus repository.
   mvn deploy command is used to deploy the build artifat to the remote reposiory.
   we created a maven settings.xml file using config file provider plugin and defined credentials for nexus maven-releases and maven-snapshots repositories. 
   withMaven block sets up the maven environment that authenticates jenkins user with nexus repositories using those credentials which enables jenkins pipeline to deploy the artifact

![image](https://github.com/venkatesh-reddy679/Board_Game-CI-CD/assets/60383183/d54cd229-6150-42df-ba80-98befa6a65c9)

7. Stage to build docker image, scan the docker image using trivy and push the docker image to the docker hub repositoty. Create a global credential of type username and password in the jenkins server with docker username and password for authentication with docker hub. 

![image](https://github.com/venkatesh-reddy679/Board_Game-CI-CD/assets/60383183/d903eea9-2df1-4d65-a418-60f8f23b3b11)
![image](https://github.com/venkatesh-reddy679/Board_Game-CI-CD/assets/60383183/3332240f-c362-46e7-8479-6e15e5d17c1d)




   
   



   


   


   














 


   

