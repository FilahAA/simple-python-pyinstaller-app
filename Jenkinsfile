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
        archiveArtifacts artifacts: 'dist/add2vals'
        sh "docker run --rm -v /var/jenkins_home/workspace/submission-cicd-pipeline-filahaditia/sources:/src cdrx/pyinstaller-linux:python2 'rm -rf build dist'"
    }
}