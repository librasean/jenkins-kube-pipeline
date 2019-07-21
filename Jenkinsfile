podTemplate(
    label: 'mypod',
    inheritFrom: 'default',
    containers: [
        containerTemplate(
            name: 'docker',
            image: 'docker:18.02',
            ttyEnabled: true,
            command: 'cat'
        )
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
        def repository = 'myalpine'
        stage ('Docker') {
            container ('docker') {
                withCredentials([[$class          : 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub',
                                        usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
                      sh "docker login -u ${env.USERNAME} -p ${env.PASSWORD} ${config.container_repo.host}"
                      sh "docker build -t ${repository}:${commitId} ."
                      sh "docker push ${repository}:${commitId}"
                }
            }
        }
    }
}
