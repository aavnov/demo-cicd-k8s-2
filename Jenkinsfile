podTemplate(containers: [
  containerTemplate(name: 'maven', image: 'maven:3.6.3-adoptopenjdk-11-openj9', ttyEnabled: true, command: 'cat'),
  containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true)
  ],
  volumes: [
      hostPathVolume(hostPath: '/var/snap/docker/2746/config/daemon.json', mountPath: '/etc/docker/daemon.json'),
      hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock'),
      hostPathVolume(hostPath: '/home/docker/repository', mountPath: '/root/.m2/repository'),
  ]) 
{
  node(POD_LABEL) 
  {
    def dockerImage
    
        stage('Package'){
        sh "git clone https://github.com/MarcoGhise/demo-cicd-k8s"
        sh "ls ~/agent/workspace/my-234/demo-cicd-k8s"
        sh "find demo-cicd-k8s"
        container('maven') {
            sh "mvn clean package -f /home/jenkins/agent/workspace/my-234/demo-cicd-k8s/pom.xml"
        }
        stage('Build image'){
        container('docker') {
        sh 'du -a /etc/docker' 
        sh "docker system prune -f"    
        dockerImage = docker.build("vasilvedev/demo-cicd-k8s-app:1.0","/home/jenkins/agent/workspace/my-234/demo-cicd-k8s")
           
        withCredentials([usernamePassword( credentialsId: 'dockerhub', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) 
        { 
          sh "echo \"${PASSWORD}\" | docker login -u ${USERNAME} --password-stdin"
          dockerImage.push() 
        }
      }
    }
    }
    
    
    stage('Deploy'){
      kubernetesDeploy configs: 'demo-cicd-k8s/demo-cicd-k8s.yml', kubeconfigId: 'MyKubeConfig'
    }
  }
}
