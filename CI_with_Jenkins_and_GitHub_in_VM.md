# CI with Jenkins and GitHub

* Start Jenkins

  ```sh
  wget http://mirrors.jenkins.io/war/latest/jenkins.war
  yum -y install java-1.8.0-openjdk-devel
  nohup java -jar jenkins.war
  ```

* Add inbound port rule: 8080

* Visit http://40.73.22.218:8080 and Unlock Jenkins with initialAdminPassword

* Install suggested plugins

* Create First Admin User

* Install Locale plugin and go to Configure System (if needed)

  ```sh
  Locale - Default Language:'en_US'
  Check 'Ignore browser preference and force this language to all users'
  ```

* Install some useful plugins: Rebuilder, Safe Restart

* Configure Global Security - Matrix-based security - Add user wluo - Grant all permissions

* Add a new user gandalf and grant all permissions except 'Administer'

* TomcatServer, turn off firewall (for non-production environment)

  ```sh
  systemctl stop firewalld && systemctl disable firewalld && yum -y install git java-1.8.0-openjdk-devel
  ```

* Create git user and ssh key, apply it in Github (SSH and GPG keys)

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
  # Add inbound port rule 8090 and visit http://tomcatserver.chinanorth2.cloudapp.chinacloudapi.cn:8090
  ```

* JenkinsServer, New node - 'TomcatServer' and check 'Permanent Agent'

  ```sh
  Remote root directory: /root/.jenkins
  Launch method: Launch agents via SSH
  Host: TomcatServer IP
  Credentials: Setup Jenkins(Username, Password), then choose 'root/******'
  Host Key Verification Strategy: Non verifying Verification Strategy
  ```

* Verify the node TomcatServer with a Freestyle project

  ```sh
  Enter an item name: just-a-test
  Restrict where this project can be run: TomcatServer
  Add build step: Execute shell
  Command: ifconfig
  # Build Now & View Console Output
  ```

* Fork a sample SpringBoot project (princeqjzh/order) in Github

* Make sure you can connect Github using SSH

  ```sh
  git config --global user.name "warrenluo" && git config --global user.email "warrenluo@126.com"
  ssh-keygen -t rsa -C "warrenluo@126.com"
  eval $(ssh-agent -s)
  ssh-add /c/Users/luo.wei/.ssh/id_rsa
  clip < /c/Users/luo.wei/.ssh/id_rsa.pub
  # Apply the SSH key in Github
  ssh -T git@github.com
  ```
  
* Clone with SSH

  ```sh
  git clone git@github.com:DevOps-Notes/order.git
  ```

* Open the project in IDEA, try build it

  ```sh
  Maven Projects - clean + CTRL + install - Run Maven Build
  ```

* MySQLServer, install MariaDB

  ```sh
  yum -y install mariadb mariadb-server
  systemctl start mariadb && systemctl enable mariadb
  mysql -e "UPDATE mysql.user SET Password=PASSWORD('Root1234') WHERE user='root'"
  mysql -e 'FLUSH PRIVILEGES'
  mysql -uroot -pRoot1234 -e "GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'Root1234' WITH GRANT OPTION"
  ```

* Add endpoint 3306 and Import database using SQLyog

* Apply the database to project in IDEA (.\src\main\resources\spring\applicationContext.xml)

  ```xml
  <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
      <property name="driverClassName" value="com.mysql.jdbc.Driver" />
      <property name="url" value="jdbc:mysql://YourDBHost:YourDBPort/order?autoReconnect=true&amp;useUnicode=true&amp;characterEncoding=UTF-8" />
      <property name="username" value="YourDBUsername" />
      <property name="password" value="YourDBPassword" />
  </bean>
  ```

* Try run the project in IDEA

  ```sh
  Maven Projects - Plugins - tomcat7 - tomcat7:run
  Visit http://localhost:8071
  ```

* Push the update to Github

  ```sh
  git add .
  git commit -m 'applicationContext.xml updated'
  git push
  ```

* Create Jenkins task preview-environment

  ```sh
  Restrict where this project can be run: TomcatServer
  Git Repository URL: git@github.com:DevOps-Notes/order.git
  Additional Behaviours - Check out to a sub-directory: order
  Build - Execute shell:
  #
  BUILD_ID=DONTKILLME
  . /etc/profile
  export PROJ_PATH=`pwd`
  export TOMCAT_APP_PATH=/root/apache-tomcat-9.0.35
  sh $PROJ_PATH/order/deploy.sh
  #

  # deploy.sh
  #!/usr/bin/env bash
  killTomcat()
  {
      pid=`ps -ef | grep tomcat | grep java | awk '{print $2}'`
      echo "Tomcat pid list: $pid"
      if [ "$pid" = "" ]
      then
        echo "No alive tomcat process"
      else
        kill -9 $pid
      fi
  }

  # Build project
  cd $PROJ_PATH/order && mvn clean install

  # Stop Tomcat
  killTomcat

  # Delete old version
  rm -f  $TOMCAT_APP_PATH/webapps/ROOT.war
  rm -rf $TOMCAT_APP_PATH/webapps/ROOT

  # Copy new version
  cp $PROJ_PATH/order/target/order.war $TOMCAT_APP_PATH/webapps/
  cd $TOMCAT_APP_PATH/webapps/ && mv order.war ROOT.war

  # Start Tomcat
  cd $TOMCAT_APP_PATH/ && sh bin/startup.sh
  ```

* Build Now and visit http://tomcatserver.chinanorth2.cloudapp.chinacloudapi.cn:8090

* Do some changes to a JSP file, then push the changes to Github and Rebuild in Jenkins.