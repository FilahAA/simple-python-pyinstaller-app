node {
    withDockerContainer('python:2-alpine'){
        stage('Build') { 
            checkout scm
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
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
        checkout scm
        sh "docker run --rm -v /var/jenkins_home/workspace/submission-cicd-pipeline-filahaditia/sources:/src cdrx/pyinstaller-linux:python2 'pyinstaller --onefile add2vals.py'"
        archiveArtifacts artifacts: "sources/dist/add2vals"
        sh "docker run --rm -v /var/jenkins_home/workspace/submission-cicd-pipeline-filahaditia/sources:/src cdrx/pyinstaller-linux:python2 'rm -rf build dist'"
        sleep time: 1, unit: 'MINUTES'
        echo 'succesfull deployed inside Jenkins'
        input message: 'Lanjutkan ke tahap Deploy to AWS ?'
        sh 'scp * ec2-user@54.254.139.194'
        sh "ssh ec2-user@54.254.139.194 'pyinstaller --onefile source/add2vals.py'"
    }
}