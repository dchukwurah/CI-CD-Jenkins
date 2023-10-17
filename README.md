# CI-CD-Jenkins Pipeline

# Creating a Jenkins Server

In this guide, we will create our own Jenkins server hosted on an EC2 instance and then use this to set up a CI/CD pipeline.

1. Firstly, create an EC2 instance on AWS that will host our Jenkins and configuring the security group rules allow SSH, HTTP and port 8080 to communicate with Jenkins.

2. Next, we can SSH into the instance and run the following commands to install Java and Jenkins and the required dependencies needed such as Java etc.

```bash

sudo apt update
sudo apt install default-jdk
java -version
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 5BA31D57EF5975CA
sudo apt update
sudo apt install jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

3. Now we have our Jenkins server up and running, we can unlock and sign in to the Jenkins UI by navigating to the server's public IP address on port 8080. 
We can unlock Jenkins by entering the 'Administrator password' by getting the password from `sudo cat /var/lib/jenkins/secrets/initialAdminPassword`


4. The next step is to install the necessary plugins to create our CI/CD pipeline we can install the suggested plugins as it provides node and git integrations which is what we will need to deploy our app.
We need to ensure these plugins are installed


   - Build Features > SSH Agent

   - Build Tools > NodeJS

   - Source Code Management > Git and GitHub


8. The Jenkins Server should be successfully set up. We can now make our CI/CD pipeline 

### Creating a job in Jenkins

**Step 1.** Log In to Jenkins: Open your web browser and navigate to your Jenkins server's URL. <br>
**Step 2.** Log in using your credentials. <br>
**Step 3.** Create a New Job: From the Jenkins dashboard, click on "New Item" or "Create New Job" on the left-hand side. <br>
**Step 4.** Enter a name for your job and select the type of job you want to create. <br>
**Step 5.** Select Freestyle project <br>
**Step 6.** Check `Discard old builds` <br>
**Step 7.** `Max # of builds to keep: 3` <br>


**Step 8.** Press ok <br>
### Configure Source Code Management: 
**Step 1.** Create a new Jenkins job or navigate to an existing one.
**Step 2.** Under "Source Code Management," select "Git."
**Step 3.** Enter your repository URL and select your GitHub credentials.
**Step 4.** Specify the branch to build (e.g., */dev). 


### Testing Jenkins - does Jenkins have the env required for our deployment?

**Step 1.** Follow the above steps <br>
**Step 2.** Navigate to `Build` <br>
**Step 3.** Select `Execute Shell` <br>
**Step 4.** Enter the commands ` cd app` <br>
**Step 5.** Enter `Save` <br>

### Triggering the job manually

**Step 1.** Locate the job by using the dropdown and select `Build now` <br>
**Step 2**. Navigate to the # use the dropdown and select `console output` <br>
**Step 3.** You will now be able to see output from the job run

### Triggering the job automatically

**Step 1.** Select your job <br>#

**Step 2.** Navigate to `Configure`<br>

**Step 3.** Navigate to `Post-build Actions` <br>

**Step 4.** Enter the name of your second job <br>

**Step 5.** Check `Trigger only if build is stable` <br>

**Step 6.** Then `Save` <br>

## Merging a branch job in Jenkins

#### Firstly follow the steps in [jenkinsCreateJob.md](jenkinsCreateJob.md) to create a new job. 
#### After running a test to test if the dev branch is properly validated.
#### It is important to test on the dev branch as we dont want to work in main or for any 
### Configure Source Code Management: 
Create a new Jenkins job or navigate to an existing one.
Under "Source Code Management," select "Git."
Enter your repository URL and select your GitHub credentials.
Specify the branch to build (e.g., */dev). 

### Build Configuration:
Configure the build steps as per your project requirements (e.g., compile code, run tests).

### Configure Git Publisher:
- Scroll down to "Post-build Actions."
- Click "Add post-build action" and select "Git Publisher."
- Check "Push Only If Build Succeeds" if you only want to push when the build is successful.
- Under "Branches," click "Add Branch" and configure as follows:
- Branch to push: Enter the name of the branch you want to push (e.g. main).
- Target remote name: e.g. "origin" (or use the remote name you have configured for GitHub).
Jenkins CI/ **CD**

#### 3rd Job
1. Firstly, we will create a new job called 'chiedozie-cd' which will be used to release our new app  from GitHub `main`<br<>0 branch to the production instance hosted by AWS.

2. Then we can go to 'Source Code Management' and specify our `main` branch to download from since this will now be updated with the tested dev changes.

3. Under 'Build Environment' we want to 'Provide Node & npm bin/ folder to PATH' and check the box for 'SSH Agent', selecting our tech254.pem SSH key which allows Jenkins access to the AWS EC2 instance via SSH.

4. Now, we can add a build step to 'Execute shell' as shown below. This will copy the app folder from the Jenkins workspace, which is from the merged main branch on GitHub, into the AWS EC2 instance, specifying the IP address of the EC2 instance to connect to. Then using the ssh command, we can remotely run commands to install and run the updated application.
```bash
rsync -avz -e "ssh -o StrictHostKeyChecking=no" app ubuntu@<ip-of-instance>:/home/ubuntu/

ssh -o "StrictHostKeyChecking=no" ubuntu@<ip-of-instance> <<EOF
	sudo apt-get update -y
    sudo apt-get upgrade -y
    sudo apt-get install nginx -y
    sudo systemctl restart nginx 
    sudo systemctl enable nginx
    
EOF
```
6. In another job we can enter another command to launch the app:
   ```bash
rsync -avz -e "ssh -o StrictHostKeyChecking=no" app ubuntu@<ip-of-instance>:/home/ubuntu/

ssh -o "StrictHostKeyChecking=no" ubuntu@<ip-of-instance> <<EOF
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
sudo apt-get install -y nodejs
cd app
sudo npm install pm2 -g
pm2 start app.js
    
EOF
```
7. Finally, we can save the job and manually build it to test t00he  if it succeeds, we can add it to the pipeline by editing our configuration for the 'chiedozie-ci-merge' job adding a 'Post-build Action' to start the 'chiedozie-cd' job, using [jenkinsMerge.md](jenkinsMerge.md).

## Checking the CI/CD Pipeline

1. Now, if we make a change in the application code, for example, editing a test in 'index.ejs' to remove the word 'Sparta' logo and push the change to the `dev` branch on the GitHub repository, our Jenkins pipeline should automatically start the build process and if all is successful and the automated tests should fail and the app wouldn't be running,
2. but if we make another change that doesn't affect the tests and push ,it will be tested and a new version will be deployed
