//Для привязки kubectl к окружению необходимо на хосте запустить:

// kubectl create clusterrolebinding serviceaccounts-cluster-admin \
//   --clusterrole=cluster-admin \
//   --group=system:serviceaccounts



podTemplate(containers: [
  containerTemplate(name: 'maven', image: 'maven:3.6.3-adoptopenjdk-11-openj9', ttyEnabled: true, command: 'cat'),
  containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true),
  containerTemplate(name: 'kubectl', image: 'lachlanevenson/k8s-kubectl:v1.14.0', command: 'cat', ttyEnabled: true)
  ],
  volumes: [
      hostPathVolume(hostPath: '/var/snap/docker/2746/config/daemon.json', mountPath: '/etc/docker/daemon.json'),
      hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock'),
      hostPathVolume(hostPath: '/home/docker/repository', mountPath: '/root/.m2/repository'),
  ]) 
{
    node(POD_LABEL) {
    def dockerImage
    //def imagename = 'vasilvedev/demo-cicd-k8s-2-app:1.0'
    
        stage('Clone repository & Build package'){
            sh "git clone https://github.com/aavnov/demo-cicd-k8s-2"
            //sh "ls ~/agent/workspace/my-345_main/demo-cicd-k8s-2"
            //sh "find demo-cicd-k8s-2"
            container('maven') {
                sh "mvn clean package -f ${env.WORKSPACE}/demo-cicd-k8s-2/pom.xml"
            }
        }
        stage('Build image'){
            container('docker') {
                sh 'du -a /etc/docker'
                sh "docker system prune -f"
                //dockerImage = docker.build imagename
                dockerImage = docker.build("vasilvedev/demo-cicd-k8s-2-app:1.0","${env.WORKSPACE}/demo-cicd-k8s-2")
           
                withCredentials([usernamePassword( credentialsId: 'dockerhub', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                   sh "echo \"${PASSWORD}\" | docker login -u ${USERNAME} --password-stdin"
                   dockerImage.push()
                }
           }
        }
    
    
        stage('Deploy'){
            container('kubectl') {
                sh 'kubectl version'
                sh 'kubectl apply  -f ./demo-cicd-k8s-2/demo-cicd-k8s.yml'
            }
        }
   }
}
