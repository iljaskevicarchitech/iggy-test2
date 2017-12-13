def podName = 'iggy-nginx-test'
def imageName = 'iggy-test-img'
def registryUrl = 'igortestacs.azurecr.io'
def registryUser = 'igortestacs'
def registryPass = 'wbc4thcq602BTdrJFZxF=sH36u3mroJC'
node {
  stage 'Checkout'

  checkout scm


  String newVersionConf = sh(returnStdout: true, script: "cat version.conf").trim()
  String gitCommitNum = sh(returnStdout: true, script: "git rev-list HEAD --count").trim()
  String gitShortId = sh(returnStdout: true, script: "git rev-parse --short HEAD").trim()
  String newLabel = newVersionConf + '.' + gitCommitNum + '-' + gitShortId

  String oldQaVersion = sh(returnStdout: true, script: "/usr/local/bin/kubectl get deployment/\${podName} --namespace=qa-env -o wide | awk '{ if (NR!=1) {print \$8}}' | cut -d ':' -f 2").trim()
  String oldProdVersion = sh(returnStdout: true, script: "/usr/local/bin/kubectl get deployment/\${podName} --namespace=prod-env -o wide | awk '{ if (NR!=1) {print \$8}}' | cut -d ':' -f 2").trim()

  stage('Dummy compile step') {
    sh "sleep 5"
  }

  stage('Dummy unit test step') {
    sh "sleep 10"
  }

  stage('Build Docker image') {
    sh "docker build -t ${registryUrl}/${imageName}:${newLabel} ."
  }

  stage('Pushing to Docker registry') {
    sh "docker login -u ${registryUser} -p ${registryPass} ${registryUrl}"
    sh "docker push ${registryUrl}/${imageName}:${newLabel} "
  }

/* Creating deployent will be handled by Helm Chart,
this Deploy stage is for testing purposes only */
  /*stage('Deploy to Kubernetes cluster') {
    sh "/usr/local/bin/kubectl create --namespace=qa-env -f test-deployment.yml"
  }*/

  stage('Update QA approval'){
    input "Update the Pod image in QA env?"
  }

  //def oldVersion = '1.7.9'
  //def newVersion = '1.9.1'
  //String newVersion = sh(returnStdout: true, script: "cat version.conf").trim()


  stage('Update Pod image on Kubernetes cluster - QA environment') {
    sh "/usr/local/bin/kubectl set image deployment/${podName} ${podName}=${registryUrl}/${imageName}:${newLabel} --namespace=qa-env"

  }

// Script to get current image version number in QA environment
// Change 'deployment/nginx-test' to 'deployments/${podName}'
// Change 'qa-env' to appropriate namespace
// sh "kubectl --kubeconfig ~/.kube/acsGenerated.config get deployment/nginx-test --namespace=qa-env -o wide | awk '{ if (NR!=1) {print $8}}' | cut -d ':' -f 2"

// Script to add a tag to an existing image
// 'iggy-test-img:v1.0' is existing image/tag
// 'iggy-test-img:hello' is new image/tag that points to existing image/tag
// docker tag iggy-test-img:v1.0 iggy-test-img:hello

// Script remove tag
// docker rmi iggy-test-img:hello


  //currentBuild.result = 'SUCCESS'

  stage('Rollback QA approval'){
    input "Roll back the Pod image in QA?"
  }

  stage('Roll Back Pod image on Kubernetes cluster - QA environment') {
    /* sh "/usr/local/bin/kubectl create -f test-deployment.yml" */
    sh "/usr/local/bin/kubectl set image deployment/${podName} ${podName}=${registryUrl}/${imageName}:${oldQaVersion} --namespace=qa-env"
  }

  stage('Dummy QA test step') {
    sh "sleep 10"
  }

  stage('Update PROD approval'){
    input "Update the Pod image in PROD env?"
  }

  stage('Update Pod image on Kubernetes cluster - PROD environment') {
    sh "/usr/local/bin/kubectl set image deployment/${podName} ${podName}=${registryUrl}/${imageName}:${newLabel} --namespace=prod-env"

  }

  stage('Rollback PROD approval'){
    input "Roll back the Pod image in PROD?"
  }

  stage('Roll Back Pod image on Kubernetes cluster - PROD environment') {
    /* sh "/usr/local/bin/kubectl create -f test-deployment.yml" */
    sh "/usr/local/bin/kubectl set image deployment/${podName} ${podName}=${registryUrl}/${imageName}:${oldQaVersion} --namespace=prod-env"
  }
}


