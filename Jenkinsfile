pipeline {
    agent any
    options {
        skipDefaultCheckout(true)
    }
    stages {
        stage('Code checkout from GitHub') {
            steps {
                script {
                    cleanWs()
                    git credentialsId: 'github-token', url: 'https://github.com/Zimski4/abcd-student.git', branch: 'main'
                }
            }
        }
        stage('[Preparation]') {
            steps {
                sh '''
                    mkdir -p results
                    chmod -R 777 results
                '''
            }
        }
        /*stage('[ZAP] Baseline passive-scan') {
            steps {
                sh '''
                    docker run --name juice-shop -d --rm \
                        -p 3000:3000 \
                        bkimminich/juice-shop
                    sleep 5
                '''
                timeout(time: 3, unit: 'MINUTES') {
                sh '''
                    docker rm -f zap || true
                    docker run --user root --name zap \
                        --add-host=host.docker.internal:host-gateway \
                        -v /home/kali/abcd-student/.zap:/zap/wrk/:rw \
                        -t ghcr.io/zaproxy/zaproxy:stable bash -c \
                        "ls -l /zap/wrk/ ; zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive.yaml" \
                        || true
                '''
                }
            }
        */}
        stage('[OSV-Scanner] Package-lock.json') {
            steps {
                script{
                    sh 'osv-scanner --lockfile package-lock.json --format json > osv_report.json || true'
                    archiveArtifacts artifacts: 'osv-report.json'
                }
            }
        }
    }
    post {
        always {
            sh '''
                docker cp zap:/zap/wrk/reports/zap_html_report.html ${WORKSPACE}/results/zap_html_report.html || true
                docker stop zap || true
                docker rm zap || true
                docker stop juice-shop || true
                docker rm juice-shop || true
            '''
            echo 'Archiving results...'
            archiveArtifacts artifacts: 'results/**/*', fingerprint: true, allowEmptyArchive: true
        }
    }
}
