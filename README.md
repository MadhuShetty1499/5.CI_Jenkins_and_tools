# 5.CI_Jenkins_and_tools
Continuous Integration with Jenkins and tools

### Scenario:
  - Agile SDLC.
  - Developers make regular code changes.
  - These commits need to be built and tested.
  - Usually build and release team will do this job or the Developer's responsibility to merge and integrate code.

### Problem:
  - In Agile SDLC, there will be frequent code changes.
  - Not so frequently the code will be tested, which will accumulate bugs and errors in the code.
  - Developers need to rework to fix these bugs and errors.
  - Manual build and release process. Inter team dependencies.

### Solution:
  - Build and test for every commit.
  - Automated process.
  - Notify for every build status.
  - Fix code if bugs or errors are found instantly rather than waiting.

### Benefits of CI pipeline:
  - Short mean time to repair.
  - Agile.
  - No human intervention.
  - Fault isolation.

### Tools used:
  1. Jenkins - CI server
  2. Git - VCS
  3. Maven - Build tool
  4. Checkstyle - Code analysis
  5. Slack - Notification
  6. Nexus - Artifact repository
  7. Sonarqube - Code analysis server
  8. AWS EC2 - Compute resource

### Flow chart
  ![Flow chart](https://github.com/MadhuShetty1499/5.CI_Jenkins_and_tools/blob/main/images/Flowchart.png)

### Steps:
  1. Login to AWS account.
  2. Create Keypair.
  3. Create Security groups:
      a. Jenkins.
      b. Nexus.
      c. Sonarqube.
  4. Create EC2 with userdata:
      a. Jenkins.
      b. Nexus.
      c. Sonarqube.
  5. Post installation:
      a. Jenkins setup and plugins.
      b. Nexus setup and repository creation.
      c. Sonarqube login test.
  6. Git:
      a. Create a Github repo and migrate the code.
      b. Integrate Github repo with VS code and test it.
  7. Build job with Nexus integration.
  8. Github webhook.
  9. Sonarqube server integration stage.
  10. Nexus Artifact upload stage.
  11. Slack notification.

### Detailed steps:
  #### Create Key pair:
  - Go to AWS EC2 => key pair => create => pem key
  ![Key pair](https://github.com/MadhuShetty1499/5.CI_Jenkins_and_tools/blob/main/images/keypair.png)

  #### Create Security group:
  - Go to AWS Ec2 => Security group:
    - Jenkins SG => Inbound rules => Allow port 8080 from anywhere and port 22 from my Ip.
    - Nexus SG => Inbound rules => Allow ports 22 and 8081 from my Ip and also port 8081 from Jenkins SG.
    - Sonarqube SG => Inbound rules => Allow ports 22 and 80 from my Ip and also port 80 from Jenkins SG.
  ![Security group](https://github.com/MadhuShetty1499/5.CI_Jenkins_and_tools/blob/main/images/securitygrp.png)
  
  #### Launch EC2 instances with user data:
  - Go to AWS EC2 => instance => launch instance:
    - Jenkins => AMI Ubuntu22(free tier) => t2.small => select key pair => Jenkins SG => Advanced settings => user data ([Jenkins](https://github.com/MadhuShetty1499/5.CI_Jenkins_and_tools/blob/main/userdata/jenkins-setup.sh)) => launch.
    - Nexus => AMI CentOS 9(free tier) => t2.medium => select key pair => Nexus SG => Advanced settings => user data ([Nexus](https://github.com/MadhuShetty1499/5.CI_Jenkins_and_tools/blob/main/userdata/nexus-setup.sh)) => launch.
    - Sonarqube => AMI Ubuntu22(free tier) => t2.medium => select key pair => Sonarqube SG => Advanced settings => user data ([Sonarqube](https://github.com/MadhuShetty1499/5.CI_Jenkins_and_tools/blob/main/userdata/sonar-setup.sh)) => launch.
  - Wait for about 10 minutes to bring up instances and provisioning services.
  ![EC2](https://github.com/MadhuShetty1499/5.CI_Jenkins_and_tools/blob/main/images/ec2.png)

  Note: Those bash scripts (User data) contain installing and provisioning services. Go through the scripts and also you can install them manually by typing those commands into your respective EC2 instances.

  - Login to all instances and check the services are running:
    - to log in - $`ssh -i <keypair> <user>@<publicIP>` (user name and ssh command can be found by selecting the required instance and clicking on connect => ssh client).
    - to check the services - $`systemctl status <servicename>`.

  #### Jenkins setup:
  - Copy the public Ip of the Jenkins EC2 instance and paste it into the browser followed by the colon and its port number. $`<PublicIp>:8080`
  - For the password, copy the path shown on the login screen => Get it in Jenkin's instance $`sudo cat /var/lib/jenkins/secrets/initialAdminPassword` => paste it into login page => select install suggested plugins => create user (Make sure to remember the password when you create the user).
  - Install Plugin => Go to manage => plugins => available plugins => select Maven integration, Github integration, Nexus artifact uploader, Sonarqube scanner, Slack notification and Build timestamp => install without restart.
  ![Plugins](https://github.com/MadhuShetty1499/5.CI_Jenkins_and_tools/blob/main/images/plugins.png)

  #### Nexus setup:
  - Copy the public Ip of the Nexus EC2 instance and paste it into the browser followed by the colon and its port number. $`<PublicIp>:8081`
  - Sign in (User - admin) => For the password, copy the path shown on the login screen => Get it in Nexus's instance $`sudo cat /opt/nexus/sonatype-work/nexus3/admin.password` => paste it into login page=> set new password (Make sure to remember/save) => disable anonymous access.
  - Go to server admin => repositories => Create repo:
    a. maven2(hosted) => vprofile-release (To store artifact) => create.
    b. maven2(proxy) => vpro-maven-central (To download dependencies from maven) => url `https://repo1.maven.org/maven2/` => create.
    c. maven2(hosted) => vprofile-snapshot (For snapshot) => version policy - Snapshot => create.
    d. maven2(group) => vprofile-maven-group => add all the other 3 repo.
  ![Nexus repo](https://github.com/MadhuShetty1499/5.CI_Jenkins_and_tools/blob/main/images/nexus_repo.png)

  #### Sonarqube setup:
  - Copy the public Ip of the Sonarqube EC2 instance and paste it into the browser followed by the colon and its port number. $`<PublicIp>:80`
  - Login (User - admin and password - admin).

  #### Git:
  - Fork the repo in Github (https://github.com/MadhuShetty1499/5.CI_Jenkins_and_tools.git).
  - Open gitbash => configure github username and email:
    a. Email $`git config --global user.email "<email>"`.
    b. Username $`git config --global user.name "<username>"`.
    c. Check if it is configured correctly $`cat .gitconfig`.
  - Create SSH keygen $`ssh-keygen`.
  - Copy public key $`cat ~/.ssh/id_rsa_pub` and paste it in Github => settings => SSH key => new => give title => paste the key => add.
  - In VS code => source control => clone the repo using ssh (copy the ssh url of the repo in Github and paste in VS code) => select ci-jenkins branch.

  #### Build job with Nexus Integration:
  - In Jenkins => manage => tools => add JDK => uncheck install automatically:
    a. OracleJDK11 => add its path `/usr/lib/jvm/java-1.11.0-openjdk-amd64` which is in Jenkin's server.
    b. OracleJDK8 => add its path `/usr/lib/jvm/java-1.8.0-openjdk-amd64`. If there is no JDK8 installed in Jenkin's server, install it $`sudo apt update && sudo apt install openjdk-8-jdk -y` and update its path.
  - Also, add Maven => MAVEN3 => install automatically => latest version 3 => save.
  - Go to manage => credentials => system => global => add => username and password of Nexus => ID - nexuslogin => save
  - In VS code => ci-jenkins branch => Jenkinsfile => write the pipeline as in [Nexus_integration](https://github.com/MadhuShetty1499/5.CI_Jenkins_and_tools/blob/main/stages/nexus_integration.txt).
  - Commit and push to Github.
  - Create a job in Jenkins => pipeline => select pipeline from SCM => add Github repo URL => add credentials => SSH => username - git => ID - githublogin => Copy private key from gitbash $`cat ~/.ssh/id_rsa` => paste => add => select credential => its displays error => login to Jenkins server => switch to root user => switch to jenkins user => $`git ls-remote -h <Github repo URL> HEAD` (It stores the identity and next time when you login it won't ask yes/no prompt) => now in Jenkins change the credentials and once again select back the git credential and the error is sorted out => branch */ci-jenkins => file - Jenkinsfile => save and build.
  Note: If the job fails due to invalid variables, go to settings.xml and pom.xml and change the variables with underscore instead of hyphen (because, Jenkins doesn't support hyphens used in variables), commit, push and build it.

  #### Github web hook (Whenever there is a commit, build automatically triggers):
  - Check if Jenkins Security group is allowing port 8080 from anywhere. Because Github webhook triggers Jenkins build when there is a commit. Git is outside our network, so the connection is to be allowed from anywhere.
  - In Github => repo settings => webhook => add => url - `http://<PubIp of jenkins>:8080/github-webhook/` => content type - Json => add => check in recent deliveries => make sure it has green checkmark if it is properly done.
  - In Jenkins job => configure => triggers => check Github webhook trigger => save.
  - Check the webhook trigger by adding additional stages in Jenkinsfile from [Github_webhook_integration](https://github.com/MadhuShetty1499/5.CI_Jenkins_and_tools/blob/main/stages/gitwebhook_integration.txt).
  - Commit and push the changes and the jobs get triggered automatically.

  #### Code analysis with Sonarqube:
  - In Jenkins => manage => tools => Sonarqube scanner => sonarscanner => version 4.7 => save.
  - Manage => configure system => Sonarqube server => check environment variable => sonarserver => url - `http://<privIp>:80` => add token => in Sonarqube server (Browser) => my account => security => generate token => copy and paste in Jenkins credentials as secret text => ID - sonartoken => add => select the token => save.
  - Add Environment variable and Sonar Analysis stage in pipeline which is in [Sonarqube_integration](https://github.com/MadhuShetty1499/5.CI_Jenkins_and_tools/blob/main/stages/sonarqube_integration.txt).
  - Commit and push. Check in Sonarqube server.

  #### Sonarqube quality gates:
  - In Sonarqube server => quality gate => create => vprofileQG => add => condition => bugs greater than 50 => add.
  - In project settings => select quality gate.
  - In webhooks => create => jenkinswebhook => url - `http://<privIp of jenkins>:8080/sonarqube-webhook/`.
  - Add Quality gate stage in pipeline which is in [Qualitygate_integration](https://github.com/MadhuShetty1499/5.CI_Jenkins_and_tools/blob/main/stages/qualitygate_integration.txt).
  - Commit and push. If the bugs are greater than 50, the pipeline will fail.

  #### Publish artifact to Nexus repo:
  - Go to Jenkins => manage => configure => Build timestamp => set pattern - yy-MM-dd_HHmm => save.
  - Add Upload Artifact stage in the pipeline which is in [Artifact_uploader](https://github.com/MadhuShetty1499/5.CI_Jenkins_and_tools/blob/main/stages/artifact_uploader.txt).
  - Commit and push. Check the artifact in the Nexus repo.

  #### Slack notification:
  - In the browser search login slack => create workspace - vprofilecicd => work - devopscicd => skip the add members step => you can install app or continue in browser => add channel => jenkinscicd => save.
  - In browser search add apps to slack => search Jenkins CI => add => choose channel Jenkinscicd => add => jenkins integration => copy the token => save.
  - In Jenkins => manage => configure => slack => workspace - vprofilecicd => credential => secret text => paste the token => ID - slacktoken => channel - #jenkinscicd => test connection (if it fails, then re-do the steps once again) => save.
  - Add Color map variable at top outside of pipeline and post stage at bottom of pipeline which is in [Slack_integration](https://github.com/MadhuShetty1499/5.CI_Jenkins_and_tools/blob/main/stages/slack_integration.txt).
  - Commit and push. Check the notification in the slack. If the build fails, the notification appears red or else green.
  ![Slack](https://github.com/MadhuShetty1499/5.CI_Jenkins_and_tools/blob/main/images/slack.png)

### Credits:
  - https://github.com/hkhcoder/vprofile-project
