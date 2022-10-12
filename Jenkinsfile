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
                archiveArtifacts artifacts: "sources/dist/add2vals"
                sshagent(credentials: ['ubuntu-app-server']) {
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@ec2-54-254-139-194.ap-southeast-1.compute.amazonaws.com "rm -rf python"'
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@ec2-54-254-139-194.ap-southeast-1.compute.amazonaws.com "mkdir -p python"'
                    sh "scp ${env.BUILD_ID}/sources/dist/add2vals ubuntu@ec2-54-254-139-194.ap-southeast-1.compute.amazonaws.com:/home/ubuntu/python"
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@ec2-54-254-139-194.ap-southeast-1.compute.amazonaws.com "cd python / sudo chmod a+x add2vals"'
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@ec2-54-254-139-194.ap-southeast-1.compute.amazonaws.com "./add2vals"'
                }
                sh "docker run --rm -v $VOLUME $IMAGE 'rm -rf build dist'"
            }
        }
        sleep time: 1, unit: 'MINUTES'
        echo 'deploye succesfull'
    }
}