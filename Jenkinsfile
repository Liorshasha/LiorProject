def appname = "hello-newapp"
def repo = "liorshasha"  // Replace with your DockerHub username
def appimage = "docker.io/${repo}/${appname}"
def apptag = "${env.BUILD_NUMBER}"

podTemplate(cloud: 'kubernetes', containers: [
    containerTemplate(
        name: 'jnlp', 
        image: 'jenkins/inbound-agent:latest'
    ),
       containerTemplate(
        name: 'trivy',
        image: 'aquasec/trivy:latest',
        command: 'sleep',
        args: '99d'
    ),
    containerTemplate(
        name: 'bandit',
        image: 'cytopia/bandit:latest', 
        command: 'sleep',
        args: '99d'
    ),
     containerTemplate(
        name: 'docker', 
        image: 'docker:26-dind', // Use the latest stable DinD image
        privileged: true,      // Essential for Docker daemon to run
        args: '--storage-driver=vfs' // VFS is safest for K8s, though slower
    )], 
  volumes: [
    emptyDirVolume(mountPath: '/var/lib/docker', memory: false) // Q: Why do we need this volume?
  ]) {
    node(POD_LABEL) {
        stage('chackout scm') {
            container('jnlp') {
            sh '/usr/bin/git config --global http.sslVerify false'
	    checkout scm
          }
        } // end chackout
        stage("linting")
        {parallel(
            'Flake8 check':{echo "Flake8 check"},
             'ShellCheck':{echo "ShellCheck"}
        )     
        stage("Security Scanning")
        {parallel(
            'Trivy for Docker':{
                container('trivy'){ 
                    echo "trivy runnig"  
                    sh "trivy fs . --exit-code 0"               
                }
            },
            'bandit scan':{
                container('bandit'){ 
                    echo "Bandit runnig"  
                    sh "bandit -r ."} 
            }
            
        )
        
        }
 container('docker') {
       stage('docker build') {
           
              echo "Building docker image..."
              sh "docker build -t ${appimage}:${apptag} ."
            }
       stage('Login and Push') {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_TOKEN'
                )]) {
                sh """
                    echo $DOCKER_TOKEN | docker login -u $DOCKER_USER --password-stdin
                    docker push $appimage:$apptag
                """
                    
                }
            }
             stage('install helm') {
             sh """
            apk add --no-cache curl bash
            curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-4
            chmod 700 get_helm.sh
            ./get_helm.sh
            helm template ${appname} ./chart
             """
        } //end hello
    }
}
  }
  }
