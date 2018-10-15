node {
    def app

    stage('Clone repository') {
        /* Let's make sure we have the repository cloned to our workspace */

        checkout scm
    }

    stage('Inject Fargate Defender') {
        sh 'cat Dockerfile'
        sh 'curl -u $TL_USER:$TL_PASSWORD -o fargate_defender.tar.gz https://$TL_CONSOLE/api/v1/images/twistlock_defender_server.tar.gz'   
        sh 'tar xvfz fargate_defender.tar.gz'
        sh 'curl -u $TL_USER:$TL_PASSWORD -o Fargate_Dockerfile https://$TL_CONSOLE/api/v1/defenders/dockerfile?consoleaddr=$TL_CONSOLE&imagename=neilcar/fargate_demo'
        sh 'head -n -1 Fargate_Dockerfile > temp.txt ; mv temp.txt Fargate_Dockerfile'
        sh 'cat ENTRYPOINT >> Fargate_Dockerfile'
        sh 'cat Fargate_Dockerfile >> Dockerfile'
        sh 'cat Dockerfile'
    }

    stage('Build image') {
        /* This builds the actual image; synonymous to
         * docker build on the command line */

        app = docker.build("neilcar/fargate_demo")
        echo app.id
        // echo app.parsedId
    }



    stage('Scan image') {
        twistlockScan ca: '', cert: '', compliancePolicy: 'warn', dockerAddress: 'unix:///var/run/docker.sock', ignoreImageBuildTime: true, image: 'neilcar/fargate_demo:latest', key: '', logLevel: 'true', policy: 'warn', requirePackageUpdate: true, timeout: 10
    }
    
    stage('Publish scan results') {
        twistlockPublish ca: '', cert: '', dockerAddress: 'unix:///var/run/docker.sock', ignoreImageBuildTime: true, image: 'neilcar/fargate_demo:latest', key: '', logLevel: 'true', timeout: 10
    }

    stage('Push image') {
        /* Finally, we'll push the image with two tags:
         * First, the incremental build number from Jenkins
         * Second, the 'latest' tag.
         * Pushing multiple tags is cheap, as all the layers are reused. */
        docker.withRegistry('http://demo-neil-lab-twistlock-com:5000') {
            app.push("${env.BUILD_NUMBER}")
            app.push("latest")
        }
    }
}
