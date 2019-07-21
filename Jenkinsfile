podTemplate(
    label: 'mypod',
    inheritFrom: 'default',
    containers: [
        containerTemplate(
            name: 'docker',
            image: 'docker:18.02',
            ttyEnabled: true,
            command: 'cat'
        ),
        containerTemplate(name: 'helm', image: 'lachlanevenson/k8s-helm:v2.6.0', command: 'cat', ttyEnabled: true),
        containerTemplate(name: 'kubectl', image: 'lachlanevenson/k8s-kubectl:v1.11.9', command: 'cat', ttyEnabled: true)
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
                    sh "kubectl get nodes"
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
