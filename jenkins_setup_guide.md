# Jenkins Setup for Ubuntu
## Setup Jenkins Coordinator
### Install Jenkins 
```bash
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee   /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]   https://pkg.jenkins.io/debian-stable binary/ | sudo tee   /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```

### Open firewall
```bash
sudo ufw allow 8080
sudo ufw status
```

### Setup
1. Go to port `http://localhost:8080`
2. Enter password from `/var/lib/jenkins/secrets/initialAdminPassword` 

### Set executers on master to 0
1. Navigate to dashboard
2. Click build executor
3. Configure master executers to be 0


## Setup agent node
### Install Java
```bash
sudo apt update 
sudo apt install openjdk-8-jdk 
```

### Create Jenkins user
```bash
sudo useradd -m -d /var/lib/jenkins jenkins
sudo su jenkins
mkdir /var/lib/jenkins/jenkins_slave
mkdir /var/lib/jenkins/.ssh
touch /var/lib/jenkins/.ssh/known_hosts
touch /var/lib/jenkins/.ssh/authorized_keys
chmod 600 /var/lib/jenkins/.ssh/authorized_keys
chmod 700 /var/lib/jenkins/.ssh
ssh-keygen -t rsa
```

### Authorise Controller to SSH to an Agent
Create an RSA SSH key on the controller node and view it:
```bash
sudo su jenkins
ssh-keygen -t rsa
vim ~/.ssh/id_rsa.pub
```
Add the public key to the `~/.ssh/authorized_keys` file of the jenkins user on the agent node:
```bash
sudo su jenkins
vim /var/lib/jenkins/.ssh/authorized_keys
```

### Configure Agent in Jenkins
1. User is `jenkins`
2. Root directory `/var/lib/jenkins`
3. Create a credential, specifying `jenkins` as the user and provide the private key of the controller machine in `/var/lib/jenkins/.ssh/id_rsa`.

## Install plugins (OFFLINE)
*[Plugin Installation Manager Tool](https://github.com/jenkinsci/plugin-installation-manager-tool)*

1. On a machine with internet access, run the following commands to download plugins, amending the plugins list as appropiate (a YAML config file can also be specified):
```bash
mkdir -p ~/plugins
cd ~/plugins
wget https://github.com/jenkinsci/plugin-installation-manager-tool/releases/download/2.12.13/jenkins-plugin-manager-2.12.13.jar
java -jar jenkins-plugin-manager-*.jar --plugin-download-directory . --plugins kubernetes
```
2. Copy these files to the Jenkins controller in the plugin directory:
```bash
sudo chown jenkins:jenkins *.jpi
sudo cp -n *.jpi /var/lib/jenkins/plugins`
```
3. Restart Jenkins by running `<jenkins_url>:<port>/restart`
