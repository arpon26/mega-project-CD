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

                    // MySQL rollout
                    sh 'kubectl rollout status deployment/mysql -n webapps --timeout=300s'

                    // --- Diagnostics (runs even if bankapp is broken) ---
                    sh '''
                        set +e
                        echo "==================== PODS ===================="
                        kubectl get pods -n webapps -o wide

                        echo "==================== SERVICES ===================="
                        kubectl get svc -n webapps

                        echo "==================== PVC ===================="
                        kubectl get pvc -n webapps

                        echo "==================== BANKAPP DEPLOYMENT ===================="
                        kubectl describe deployment bankapp -n webapps

                        echo "==================== BANKAPP POD DETAILS ===================="
                        kubectl describe pods -n webapps -l app=bankapp

                        echo "==================== RECENT EVENTS ===================="
                        kubectl get events -n webapps --sort-by=.lastTimestamp | tail -50

                        echo "==================== BANKAPP LOGS (current) ===================="
                        kubectl logs -n webapps -l app=bankapp --tail=100 --all-containers=true

                        echo "==================== BANKAPP LOGS (previous crash) ===================="
                        kubectl logs -n webapps -l app=bankapp --tail=100 --all-containers=true --previous

                        echo "==================== END OF DIAGNOSTICS ===================="
                    '''

                    // bankapp rollout (this is what fails)
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
