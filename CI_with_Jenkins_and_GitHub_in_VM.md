# CI with Jenkins and GitHub

* Start Jenkins

  ```sh
  wget http://mirrors.jenkins-ci.org/war/latest/jenkins.war
  yum -y install java
  java -jar jenkins.war
  ```

* Add inbound port rules: 8080

* Visit http://40.73.22.218:8080 and Unlock Jenkins with initialAdminPassword

* Install suggested plugins

* Create First Admin User

* Install Locale plugin and go to Configure System

  ```sh
  Locale - Default Language:'en_US'
  Check 'Ignore browser preference and force this language to all users'
  ```

* Install some more useful plugins: Rebuilder, Safe Restart

* Configure Global Security - Matrix-based security - Add user wluo - Grant all permissions

* Add a new user gandalf and grant all permissions except 'Administer'

* TomcatServer, turn off firewall (for non-production environments)

  ```sh
  systemctl stop firewalld && systemctl disable firewalld && yum -y install java git
  ```

* Create git user and ssh key, apply it in 'SSH and GPG keys' of Github

  ```sh
  git config --global user.name "warrenluo" && git config --global user.email "warrenluo@126.com"
  ssh-keygen -t rsa -C "warrenluo@126.com" && vim /root/.ssh/id_rsa.pub
  ```

* Connect Github with ssh key

  ```sh
  ssh git@github.com
  ```

* Install Maven

  ```sh
  wget https://mirror.bit.edu.cn/apache/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.zip
  unzip -q apache-maven-3.6.3-bin.zip
  vim /etc/profile
  #
  export MAVEN_HOME=/root/apache-maven-3.6.3
  export PATH=$MAVEN_HOME/bin:$PATH
  #
  . /etc/profile
  mvn -version
  ```

* Install Tomcat

  ```sh
  wget https://mirror.bit.edu.cn/apache/tomcat/tomcat-9/v9.0.35/bin/apache-tomcat-9.0.35.zip
  unzip -q apache-tomcat-9.0.35.zip
  cd apache-tomcat-9.0.35 && chmod a+x -R *
  vim conf/server.xml
  #
  <Connector port="8090"
  #
  bin/startup.sh
  ps -ef | grep tomcat
  # Add inbound port rules 8090 and visit http://tomcatserver.chinanorth2.cloudapp.chinacloudapi.cn:8090
  ```

* JenkinsServer, New node - 'Test Env'' and check 'Permanent Agent'

  ```sh
  Remote root directory: /root/.jenkins
  Launch method: Launch agents via SSH
  Host: IP of TomcatServer
  Credentials: Username, Password
  Host Key Verification Strategy: Non verifying Verification Strategy
  ```

* Verify the Test Env with a Freestyle project

  ```sh
  Enter an item name: Test Env
  Restrict where this project can be run: Test Env
  Add build step: Execute shell
  Command: ifconfig
  # Build Now & View Console Output
  ```

  