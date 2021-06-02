try{
    node{
        def mavenHome
        def mavenCMD
        def docker
        def dockerCMD
        def tagName = "2.0"
        def scannerHome
        
        
        stage('Clean up'){
            echo "Cleaning up the workspace..."
            cleanWs()
        }
        stage('Preparation'){
            echo "Preparing the Jenkins environment with required tools..."
            mavenHome = tool name: 'maven3', type: 'maven'
            mavenCMD = "${mavenHome}/bin/mvn"
            docker = tool name: 'Dockertool', type: 'org.jenkinsci.plugins.docker.commons.tools.DockerTool'
            dockerCMD = "${docker}/bin/docker"
          scannerHome = tool 'SonarCubeScanner'
          ORGANIZATION = "gunjan-dev"
          PROJECT_NAME = "batch10"
          
         }
        
        stage('git checkout'){
            echo "Checking out the code from git repository..."
              git 'https://github.com/Gunjan-dev/batch10-1.git'
              
        }
        
        stage('Build, Test and Package'){
            echo "Building the addressbook application..."
            sh "${mavenCMD} clean package"
            sh "${mavenCMD} test site"
        }
        
        stage('Sonar Scan'){
            echo "Scanning application for vulnerabilities..."
            
             withSonarQubeEnv('SonarCubeServer') {
         
              sh "${scannerHome}/bin/sonar-scanner -Dsonar.organization=$ORGANIZATION \
              -Dsonar.java.binaries=. \
               -Dsonar.projectKey=$PROJECT_NAME \
               -Dsonar.sources=."
                
            }
           
        }
        
        stage('publishHtml"'){
            
        
          publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '/var/lib/jenkins/workspace/casestudy-gunjan/target/site', reportFiles: 'surefire-report.html', reportName: 'HTML Report', reportTitles: ''])
        }
        
        stage('Build Docker Image'){
        
            echo "Building docker image for test application ..."
  
            sh "${dockerCMD} build -t gunmail88/gunrepo:$tagName ."
         
        }
        
        stage("Push Docker Image to Docker Registry"){
            echo "Pushing image to docker hub"
            withCredentials([string(credentialsId: 'dockerhubPwd', variable: 'dockerHubPwd')]) {
    
              sh "${dockerCMD} login -u gunmail88 -p ${dockerHubPwd}"
              sh "${dockerCMD} push gunmail88/gunrepo:$tagName"
            }
        }
        
        
        stage('Deploy Application'){
            echo "Installing desired software.."
            echo "Bring docker service up and running"
            echo "Deploying addressbook application"
          
             ansiblePlaybook  credentialsId: 'sshAnsible', disableHostKeyChecking: true, installation: 'Ansible', inventory: '/etc/ansible/hosts', playbook: 'config-deploy-playbook.yml'
        }
        
        stage('set success build status for email'){
            
            currentBuild.result = 'SUCCESS'
            
       //     mail bcc: '', body: 'Your build is successful. ', cc: '', from: '', replyTo: '', subject: 'Build Success', to: 'himanshu321988@gmail.com'
            
        }
       
    }
}

catch(Exception err){
    echo "Exception occured..."
    currentBuild.result="FAILURE"
            
    // setting to send an failure email notification to the user.
}
finally {
    (currentBuild.result!= "ABORTED") && node("master") {
        echo "finally gets executed and end an email notification for every build"
        
        if (currentBuild.result == "SUCCESS"){
        mail bcc: '', body: 'Your build is Successful. ', cc: '', from: '', replyTo: '', subject: 'Build Result', to: 'himanshu321988@gmail.com'
        }
        
        if (currentBuild.result == "FAILURE"){
        mail bcc: '', body: 'Your build is not Successful. ', cc: '', from: '', replyTo: '', subject: 'Build Result', to: 'himanshu321988@gmail.com'
        }
        
    }
    
}
