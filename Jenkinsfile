pipeline {
    agent any

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: 'git', url: 'https://github.com/arpon26/mega-project-CD.git'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'k3s', contextName: '', credentialsId: 'k8-token',
                               namespace: 'webapps', restrictKubeConfigAccess: false,
                               serverUrl: 'https://192.168.68.56:6443') {
                    sh 'kubectl apply -f Manifest/manifest.yaml'
                    sh 'kubectl rollout status deployment/mysql -n webapps --timeout=300s'
                    sh 'kubectl rollout status deployment/bankapp -n webapps --timeout=400s'
                }
            }
        }

        stage('Verify') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'k3s', contextName: '', credentialsId: 'k8-token',
                               namespace: 'webapps', restrictKubeConfigAccess: false,
                               serverUrl: 'https://192.168.68.56:6443') {
                    sh 'kubectl get pods,svc,pvc -n webapps'
                }
            }
        }
    }
}
