podTemplate(
    label: 'mypod',
    inheritFrom: 'default',
    serviceAccount: 'jx-jenkins',
    containers: [
        containerTemplate(name: 'docker', image: 'docker:stable', command: 'cat', ttyEnabled: true),
        containerTemplate(name: 'helm', image: 'lachlanevenson/k8s-helm:v3.0.0-alpha.1', command: 'cat', ttyEnabled: true),
        containerTemplate(name: 'kubectl', image: 'lachlanevenson/k8s-kubectl:v1.15.1', command: 'cat', ttyEnabled: true)
    ],
    volumes: [
            hostPathVolume(
                hostPath: '/var/run/docker.sock',
                mountPath: '/var/run/docker.sock'
            )
    ]
) {
    node('mypod') {
        def commitId
        stage ('Extract') {


            checkout scm
            commitId = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
        }
        def repository = 'nelson1/myalpine'
        stage ('Docker') {
            container ('docker') {
                withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub',
                                        usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
                      sh "docker login -u ${env.USERNAME} -p ${env.PASSWORD}"
                      sh "docker build -t ${repository}:${commitId} ."
                      sh "docker push ${repository}:${commitId}"
                }
            }
        }
        stage('Deliver for development') {
            if (env.BRANCH_NAME == 'master') {
                container('kubectl') {
                    println "checking kubectl connnectivity to the API"
                    sh "kubectl get po -n ops"

                }
                container('helm') {
                    def pwd = pwd()
                    def chart_dir = "${pwd}/test"
                    println "initiliazing helm client"
                    sh "helm init"
                    println "checking client/server version"
                    sh "helm version"
                    println "Running deployment"
                    sh "pwd"
                    sh "ls -la"
                    sh "ls -la test"
                    // reimplement --wait once it works reliable
                    sh "helm upgrade --install test ${chart_dir} --set imageTag=latest,replicas=1 --namespace=letters-dev"

                    // sleeping until --wait works reliably
                    sleep(20)

                    echo "Application test successfully deployed. Use helm status test to check"
                }
            }
            if (env.BRANCH_NAME == 'development') {
                container('kubectl') {
                    sh "kubectl get pods --all-namespaces"
                }
            }
        }
    }
}
