podTemplate(
    label: 'jenkins-build',
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
    node('jenkins-build') {
        def commitId
        stage ('Extract') {
            checkout scm
            commitId = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
            println "DTR ==> ${env.DTR}"
        }
        def registry = env.DTR
        def repository = env.DTR + "/testorg/" + env.JOB_BASE_NAME
        stage ('Docker') {
            container ('docker') {

                withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dtr',
                                        usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
                      sh "docker login -u ${env.USERNAME} -p ${env.PASSWORD} ${registry}"
                      sh "docker build -t ${repository}:${commitId} ."
                      sh "docker push ${repository}:${commitId}"
                }
            }
        }
        stage('Deliver for development') {
            def namespace;
            if (env.BRANCH_NAME == 'master') {
                namespace = "letters-preprod"
            }
            if (env.BRANCH_NAME == 'development' || env.BRANCH_NAME  =~ "PR-*") {
                namespace = "letters-dev"
            }
            container('helm') {
                def pwd = pwd()
                def chart_dir = "${pwd}/test"
                println "initializing helm client"
                sh "helm init"
                println "checking client/server version"
                sh "helm version"
                println "Running deployment"
                // reimplement --wait once it works reliable
                sh "helm upgrade --install test ${chart_dir} --set imageTag=latest,replicas=1 --namespace=${namespace}"

                // sleeping until --wait works reliably
                sleep(20)

                echo "Application test successfully deployed. Use helm status test to check"
            }
        }
    }
}
