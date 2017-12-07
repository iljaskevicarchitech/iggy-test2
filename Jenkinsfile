node {
  stage 'Checkout'

  checkout scm

  stage 'Deploy to Kubernetes cluster'
  sh "kubectl create -f test-deployment.yml"
}
