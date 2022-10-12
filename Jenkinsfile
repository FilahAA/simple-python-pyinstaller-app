node {
    withDockerContainer('python:2-alpine'){
        stage('Build') { 
            checkout scm
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
            stash(name: 'compiled-results', includes: 'sources/*.py*')
        }
    }
    withDockerContainer('qnib/pytest'){
        stage('Test') { 
            checkout scm
            sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
            junit 'test-reports/results.xml' 
        }
    }
    stage('Deploy') { 
        input message: 'Lanjutkan ke tahap Deploy ?'
        checkout scm
        withEnv(['VOLUME="${PWD}/sources:/src"' , 'IMAGE="cdrx/pyinstaller-linux:python2"']) {
            dir(path: env.BUILD_ID) { 
                    unstash(name: 'compiled-results')
                    sh "docker run --rm -v $VOLUME $IMAGE 'pyinstaller --onefile add2vals.py'"
            }
            if (currentBuild.result == null || currentBuild.result == 'SUCCESS') { 
                archiveArtifacts artifacts: "${PWD}/${env.BUILD_ID}/artifact/sources/dist/add2vals"
                sh "scp -i app-server.pem ${PWD}/${env.BUILD_ID}/artifact/sources/dist/add2vals ubuntu@54.254.139.194:/"
                sh "ssh ubuntu@54.254.139.194 'sudo chmod a+x add2vals && ./add2vals'"
                sh "docker run --rm -v $VOLUME $IMAGE 'rm -rf build dist'"
            }
        }
        sleep time: 1, unit: 'MINUTES'
        echo 'deploye succesfull'
    }
}