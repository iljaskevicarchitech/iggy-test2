node {
  stage 'Checkout'

  checkout scm

  stage 'Deploy to Kubernetes cluster'
  sh "/usr/local/bin/kubectl create -f test-deployment.yml"
}
