# CI with Jenkins and GitHub

* Start Jenkins with Docker

  ```sh
  docker pull jenkinsci/jenkins
  docker run --name jenkins-svr -p 8080:8080 -v D:\temp\jenkins_home:/var/jenkins_home jenkinsci/jenkins
  ```

* Switch to local repo

  ```sh
  sed -i 's/http:\/\/www.google.com/https:\/\/www.baidu.com/g' default.json
  sed -i 's/http:\/\/updates.jenkins-ci.org\/download/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g' default.json
  ```

* Restart Jenkins

  ```sh
  docker restart **
  ```

* Visit http://127.0.0.1:8080 and Unlock Jenkins with initialAdminPassword
* Select plugins to install and choose 'None'
* Create First Admin User and Restart Jenkins

* Visit http://127.0.0.1:8080/login

