# CI-CD-Jenkins

- Set-up an EC2 instance on VPC (Go to for VPC guide)
- For the new security group we need to select ports 22, 8080, and 80
- Connect to Instance with SSH.
- Code to run on Ubuntu instance:
i. sudo apt update -y
ii. sudo apt install default-jdk -y
iii. java -version
iv. wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
v. sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 5BA31D57EF5975CA
vi. sudo apt update
vii. sudo apt install jenkins -y
viii. sudo systemctl start jenkins
ix. sudo systemctl enable jenkins
x. sudo cat /var/lib/jenkins/secrets/initialAdminPassword
Use PublicIP and port 8080
Using your Initial Admin Password paste it in
Set-up Plug-ins including Access to GitHub, SSH Agent, Git Publisher.



what is IAAC and configuration management orchestration
