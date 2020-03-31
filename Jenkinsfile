def SLN_PATH = "web01/web01.csproj"
def TEST_PATH = "test/test.csproj"
def SRC_FOLDER = "web01"

properties([
    disableConcurrentBuilds(),
    disableResume(),
])

podTemplate(inheritFrom: 'jnlp-pod', containers: [
    containerTemplate(name: 'kaniko', image: 'gcr.io/kaniko-project/executor:debug', alwaysPullImage: false, ttyEnabled: true, command: '/busybox/cat'),
    containerTemplate(name: 'gitversions', image: 'gittools/gitversion', ttyEnabled: true, command: 'cat'),
    containerTemplate(name: 'dotnet-sdk', image: 'mcr.microsoft.com/dotnet/core/sdk:3.1-alpine', ttyEnabled: true, command: 'cat'),
  ]) {

    node(POD_LABEL) {
        ansiColor('xterm') {
            stage('checkout') {
                checkout scm
            }

            stage('gitversions') {
                container('gitversions') {
                    sh """
                    /tools/dotnet-gitversion web01 -output buildserver
                    """
                }
            }

            stage('Dotnet build & tests') {
                container('dotnet-sdk') {
                    sh """
                    dotnet build $SLN_PATH
                    dotnet test $TEST_PATH
                    """
                }
            }

            stage('Build Image') {
                withCredentials([[$class: 'ZipFileBinding', credentialsId: 'config-json', variable: 'DOCKER_CONFIG']]) {
                    container('kaniko') {
                        sh """
                        MAJOR=\$(cat gitversion.properties | grep -w GitVersion_Major | cut -d '=' -f2-)
                        MINOR=\$(cat gitversion.properties | grep -w GitVersion_Minor | cut -d '=' -f2-)
                        COMMIT=\$(cat gitversion.properties | grep -w GitVersion_CommitsSinceVersionSource | cut -d '=' -f2-)
                        SHA=\$(cat gitversion.properties | grep -w GitVersion_ShortSha | cut -d '=' -f2-)
                        TAG=\$(echo "\$MAJOR.\$MINOR.\$COMMIT-\$SHA")

                        cd $SRC_FOLDER
                        /kaniko/executor --context dir://. --cache=false --insecure --skip-tls-verify --destination=azalaxdev/demos:web-demo-\$TAG
                        """
                    }
                }
            }
        }
    }
}
