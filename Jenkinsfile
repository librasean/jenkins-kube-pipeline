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
        containerTemplate(name: 'kubectl', image: 'lachlanevenson/k8s-kubectl:v1.4.8', command: 'cat', ttyEnabled: true)
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
            // create git envvars
            println "Setting envvars to tag container"

            sh 'git rev-parse HEAD > git_commit_id.txt'
            try {
                env.GIT_COMMIT_ID = readFile('git_commit_id.txt').trim()
                env.GIT_SHA = env.GIT_COMMIT_ID.substring(0, 7)
            } catch (e) {
                error "${e}"
            }
            println "env.GIT_COMMIT_ID ==> ${env.GIT_COMMIT_ID}"

            sh 'git config --get remote.origin.url> git_remote_origin_url.txt'
            try {
                env.GIT_REMOTE_URL = readFile('git_remote_origin_url.txt').trim()
            } catch (e) {
                error "${e}"
            }
            sh 'git rev-parse --abbrev-ref HEAD > git_branch.txt'
            try {
                env.GIT_BRANCH = readFile('git_branch.txt').trim()
            } catch (e) {
                error "${e}"
            }
            println "env.GIT_REMOTE_URL ==> ${env.GIT_REMOTE_URL}"
            println "env.BRANCH_NAME ==> ${env.GIT_BRANCH}"
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
            if (env.GIT_BRANCH == 'master') {
                container('kubectl') {
                    sh "kubectl get pods --all-namespaces"
                }
            }
        }
    }
}
