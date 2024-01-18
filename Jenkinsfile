pipeline {
  agent any
  tools {
    maven 'Maven-360'
    jdk 'JAVA-17'
  }
  stages{
    stage('Build') {
      steps {
        sh 'mvn clean package -DskipTests=true'
      }
    }
    stage('test') {
      parallel {
        stage('testA') {
          agent any
            steps{
              echo " This is test A"
              sh 'mvn test'
            }
        }
        stage('testB') {
          agent { label 'docker' }
            steps{
              echo "this is test B"
            }
        }
      }
      post {
        success {
          dir("webapp/target/") {
            stash name: "maven-build", includes: "*.war"
          }
        }
      }
    }
  
    stage('deploy_dev') {
      when { branch 'dev'
      beforeAgent true}
      agent any
        steps {
          dir("/var/www/html")
          {
            unstash "maven-build"
          }
          sh """
          cd /var/www/html/
          jar -xvf webapp.war
          """
        }
    }
    stage('deploy_qa-1') {
      when { branch 'qa-1'
      beforeAgent true}
      agent any
        steps {
          dir("/var/www/html")
          {
            unstash "maven-build"
          }
          sh """
          cd /var/www/html/
          jar -xvf webapp.war
          """
        }
    }
  
    stage('deploy_prod') {
      when { branch 'master'
      beforeAgent true}
      agent { label 'docker' }
        steps {
          timeout(time:5, unit:'DAYS'){
            input message: 'Deployment approved?'
          }
          sh """
          whoami
          pwd
          ls -lrth
          """
          dir("/var/www/html")
          {
             unstash "maven-build"
          }
          sh """
          mkdir -p /var/www/html
          cd /var/www/html/
          ls -lrth
          """
        }
    }
  }
}
