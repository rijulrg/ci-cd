node {
    def commit_id 
    stage('Preparation') {
        checkout scm 
        sh "git rev-parse --short HEAD > .git/commit-id"
        commit_id = readFile('.git/commit-id').trim() // getting commit-id for tagging build
        }
    stage('test') {
        def myTestContainer = docker.image('node:10')
        myTestContainer.pull()
        myTestContainer.inside {    
            sh 'npm install --only=dev'
            sh 'npm test'
            }
        }
    stage('sonar-scanner') {
        def sonarqubeScannerHome = tool name: 'sonar', type: 'hudson.plugins.sonar.SonarRunnerInstallation' // require sonarqube plugin
        def node_path = tool name: 'nodejs', type: 'jenkins.plugins.nodejs.tools.NodeJSInstallation' // require nodejs plugin
        withCredentials([string(credentialsId: 'sonar', variable: 'sonarLogin')]) {
            sh "${sonarqubeScannerHome}/bin/sonar-scanner -Dsonar.host.url=http://sonarqube:9000 -Dsonar.login=${sonarLogin} -Dsonar.projectName=test_app -Dsonar.nodejs.executable=${node_path}/bin/node -Dsonar.projectKey=RG"
            }
        }
    stage('docker build/push') {
        docker.withRegistry('https://index.docker.io/v1/', 'dockerhub') {
            def app = docker.build("rijulrg/test_app:${commit_id}", '.').push() // require cloudbee docker build n publish plugin
            }
        }
    stage('deployment') {
        withKubeConfig([credentialsId: 'kubernetes']) {
            sh 'cat k8s/deployment.yaml | sed "s/{{COMMIT_ID}}/$commit_id/g" | kubectl apply -f -'
            sh 'kubectl apply -f k8s/service.yaml'
            }
        }
}          