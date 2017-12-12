def podName = 'nginx-test'
def imageName = "nginx"

node {
  stage 'Checkout'

  checkout scm

  stage('Deploy to Kubernetes cluster') {
    sh "/usr/local/bin/kubectl create -f test-deployment.yml"
  }

  stage('Update approval'){
    input "Update the Pod image?"
  }

  def oldVersion = '1.7.9'
  def newVersion = '1.9.1'

  stage('Update Pod image on Kubernetes cluster') {
    /* sh "/usr/local/bin/kubectl create -f test-deployment.yml" */
    sh "/usr/local/bin/kubectl set image deployment/${podName} nginx=${podName}:${newVersion}"
  }

  stage('Rollback approval'){
    input "Roll back the Pod image?"
  }

  stage('Roll Back Pod image on Kubernetes cluster') {
    /* sh "/usr/local/bin/kubectl create -f test-deployment.yml" */
    sh "/usr/local/bin/kubectl set image deployment/${podName} nginx=${podName}:${oldVersion}"
  }
}
