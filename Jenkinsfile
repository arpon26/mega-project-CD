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
                withKubeConfig(caCertificate: '', clusterName: 'k3s', contextName: '',
                               credentialsId: 'k8-token', namespace: 'webapps',
                               restrictKubeConfigAccess: false,
                               serverUrl: 'https://192.168.68.56:6443') {

                    sh 'kubectl apply -f Manifest/manifest.yaml'

                    sh 'kubectl rollout status deployment/mysql -n webapps --timeout=300s'

                    // Diagnostics — will always run, even if bankapp is unhealthy
                    sh '''#!/bin/bash
set +e
echo "==================== PODS ===================="
kubectl get pods -n webapps -o wide
echo ""
echo "==================== SERVICES ===================="
kubectl get svc -n webapps
echo ""
echo "==================== PVC ===================="
kubectl get pvc -n webapps
echo ""
echo "==================== BANKAPP DEPLOYMENT ===================="
kubectl describe deployment bankapp -n webapps
echo ""
echo "==================== BANKAPP POD DETAILS ===================="
kubectl describe pods -n webapps -l app=bankapp
echo ""
echo "==================== RECENT EVENTS ===================="
kubectl get events -n webapps --sort-by=.lastTimestamp | tail -50
echo ""
echo "==================== BANKAPP LOGS (current) ===================="
kubectl logs -n webapps -l app=bankapp --tail=100 --all-containers=true
echo ""
echo "==================== BANKAPP LOGS (previous crash) ===================="
kubectl logs -n webapps -l app=bankapp --tail=100 --all-containers=true --previous
echo ""
echo "==================== END OF DIAGNOSTICS ===================="
exit 0
'''

                    // This is the step that fails; wrap so pipeline continues
                    catchError(buildResult: 'FAILURE', stageResult: 'FAILURE') {
                        sh 'kubectl rollout status deployment/bankapp -n webapps --timeout=400s'
                    }
                }
            }
        }

        stage('Verify') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'k3s', contextName: '',
                               credentialsId: 'k8-token', namespace: 'webapps',
                               restrictKubeConfigAccess: false,
                               serverUrl: 'https://192.168.68.56:6443') {
                    sh '''#!/bin/bash
set +e
echo "==================== VERIFY ===================="
kubectl get pods,svc,pvc -n webapps -o wide
echo ""
echo "==================== BANKAPP POD DETAILS ===================="
kubectl describe pods -n webapps -l app=bankapp
echo ""
echo "==================== BANKAPP LOGS ===================="
kubectl logs -n webapps -l app=bankapp --tail=100 --all-containers=true
exit 0
'''
                }
            }
        }
    }
}
