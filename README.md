# SonarQube

SonarQube is an open-source tool that ensures continuous code quality and security for twenty-seven (27) programming languages.

This tool provides a detailed report of bugs, code smells, vulnerabilities, code duplications, and it supports these languages by built-in rulesets and can also be extended with various plugins.

## Install SonarQube with docker-compose on an EC2 instance

In setting up your EC2 instance, the following are required:

- An Ubuntu Instance
![ubuntu instance](https://user-images.githubusercontent.com/49791498/134748422-c044089e-6ac2-4314-a606-49d87a2dd699.png)

- A t2 medium instance type
![instance type](https://user-images.githubusercontent.com/49791498/134748774-fbfda5b8-8e3b-4dcc-bddd-da3d8fe05522.png)

- Open up required ports in your VM instance security group
![image](https://user-images.githubusercontent.com/49791498/134749294-23d1ffac-ed13-4773-929f-95e07876c99f.png)

- Click ```Launch``` to spin up your instance.

- SSH into your instance.

### Installing Docker and Docker-compose

- Install docker and create an ```ubuntu``` usergroup to which we add our ```docker``` user; to enable us run docker commands without the ```sudo``` command. Confirm its installation by checking docker's status.

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update
apt-cache policy docker-ce
sudo apt-get install -y docker-ce
sudo usermod -aG docker ubuntu
sudo systemctl status docker
```

![docker status](https://user-images.githubusercontent.com/49791498/134751011-9513a161-b383-47e6-8d63-67e36b226abe.png)

- Exit the container via the ```exit``` command and log back in, to enable the docker service run without the **sudo** command.

- Install docker-compose, change the permissions and file location. Confirm its installation by checking the version

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
docker-compose --version
```

![docker-compose version](https://user-images.githubusercontent.com/49791498/134751045-af552135-5aca-406d-8b83-662cf88eb7ff.png)

### Install sonar

Install the sonar service by running the following commands:

```bash
sudo sysctl -w vm.max_map_count=262144
```

### Creating a docker-compose file

- Create a new folder and switch to it.

```bash
mkdir sonar
cd sonar
```

- Create a docker-compose.yml file to contain docker commands to install SonarQube and PostgreSQL.

```bash
nano docker-compose.yml
```

to contain:

```bash
version: '2'

services:
  sonarqube:
    image: sonarqube
    ports:
      - '9000:9000'
    networks:
      - sonarnet
    environment:
      - sonar.jdbc.username=<unique-value>
      - sonar.jdbc.password=<unique-value>
      - sonar.jdbc.url=jdbc:postgresql://db:5432/sonar
    volumes:
      - sonarqube_conf:/opt/sonarqube/conf
      - sonarqube_data:/opt/sonarqube/data
      - sonarqube_extensions:/opt/sonarqube/extensions
    ulimits:
      nofile:
       soft: 65536
       hard: 65536
  db:
    image: postgres
    networks:
      - sonarnet
    environment:
      - POSTGRES_USER=<unique-value>
      - POSTGRES_PASSWORD=<unique-value>
    volumes:
      - postgresql:/var/lib/postgresql
      # This needs explicit mapping due to https://github.com/docker-library/postgres/blob/4e48e3228a30763913ece952c611e5e9b95c8759/Dockerfile.template#L52
      - postgresql_data:/var/lib/postgresql/data

networks:
  sonarnet:
    driver: bridge

volumes:
  sonarqube_conf:
  sonarqube_data:
  sonarqube_extensions:
  postgresql:
  postgresql_data:
```

The command `Ctrl + X` and `Y`, closes the script.

- Run the following command to start up your docker-compose file

```bash
docker-compose up -d 
```

![ter1](https://user-images.githubusercontent.com/49791498/134591430-b6ca93aa-5c66-404a-b31f-98445ce64e3c.png)

- Ensure SonarQube is running by checking your logs

```bash
sudo docker-compose logs
```

**Access SonarQube UI via:**  <http://ec2-public-dns:9000/>

**[Reference](https://www.youtube.com/watch?v=-aDjIMwYy34&t=10s)**

![image](https://user-images.githubusercontent.com/49791498/134751748-a0fa2c61-aa4b-4e28-8799-c6fbef4428a4.png)

## Integrate a Jenkins pipeline to the build

- Log into your SOnarQube account with the default values of 'admin', 'admin' for both username and pasword.
![image](https://user-images.githubusercontent.com/49791498/134770038-27df4b3e-ac68-42b4-a349-a2eb5b7a4b02.png)

- Generate tokens by clicking on 'Security' under 'My Account'.
Then under 'Tokens', give it a name and click on 'Generate token'.
![image](https://user-images.githubusercontent.com/49791498/134770067-2cfc1812-2f20-4228-b228-f1df54b36e57.png)

- While SonarQube is running, then run the following commands in your VM to download and install Jenkins.

```bash
wget -q -O - https://pkg.jenkins.io/debian/jenkins-ci.org.key | sudo apt-key add -
echo deb http://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list
sudo apt-get update
sudo apt install openjdk-8-jdk
sudo apt-get install jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins
```

### Configure Jenkins

- Open up Jenkins in your browser, by typing in: `http://<your_server_public_DNS>:8080` in the search bar.

![image](https://user-images.githubusercontent.com/49791498/133862137-b0c6f9c2-46ab-45dc-a7dd-eea8b158f465.png)

- Take the steps highlighted [in this article](https://www.blazemeter.com/blog/how-to-integrate-your-github-repository-to-your-jenkins-project) to integrate your Github repository into your Jenkins pipeline.

- Type the following command to display the Administrator password:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

- On the **Customize Jenkins page**, click **install suggested plugins.**

- Once the installation is complete, create a **First Admin User**, click **Save** and **Continue**.

- On the **Manage Jenkins** page, click on **Manage Plugins**.

- Under the **Available** tab, search for **SonarQube Scanner** and install it.

### Add Tokens

- On your SonarQube Admin account, click on the **My profile** option.

- Under **Security**, generate a token under the **Generate Tokens** option.

- On Jenkins, head to the **Manage Jenkins** page and click on **Configure System**.

- Scroll down to the **SonarQube Scanner** section and fill the prompts as shown below, click **apply** and then **save**.
![image](https://user-images.githubusercontent.com/49791498/134771056-587ec28c-2b14-4b08-8fd8-fc6eb3bb3b6c.png)

- Configure your job by clicking on 'Prepare Sonarqube scanner environment'
![image](https://user-images.githubusercontent.com/49791498/134771109-bf599c67-08a7-4da6-a395-f6239276614f.png)

- Your Jenkinsfile should contain this code

```bash
node {

    def mvnHome = tool 'Maven3'
    stage ("checkout")  {
        //write pipeline code
    }

   stage ('build')  {
    sh "${mvnHome}/bin/mvn clean install -f MyWebApp/pom.xml"
    }

     stage ('Code Quality scan')  {
       withSonarQubeEnv('SonarQube') {
       sh "${mvnHome}/bin/mvn -f MyWebApp/pom.xml sonar:sonar"
        }
   }
}
```

- Further configurations depending on the programming language of choice, can be found in [this article](https://docs.sonarqube.org/latest/analysis/scan/sonarscanner-for-jenkins/).

- For building a pipeline for Flask projects, read [this article](https://dev.to/mmphego/how-i-configured-sonarqube-for-python-code-analysis-with-jenkins-and-docker-28fm)

## Creating a Terraform script

- We created the following Terraform scripts to act as place holders

```bash
main.tf
variables.tf
versions.tf
output.tf
database.tf
```

- Then run the following Terraform commands

```bash
terraform init
terraform plan
terraform apply
```
