podTemplate(
    label: 'mypod',
    inheritFrom: 'default',
    containers: [
        containerTemplate(
            name: 'docker',
            image: 'docker:stable-dind',
            ttyEnabled: true,
            command: 'cat',
            privileged: true
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
                sh "docker build -t ${repository}:${commitId} ."
                sh "docker push ${repository}:${commitId}"
            }
        }
    }
}
