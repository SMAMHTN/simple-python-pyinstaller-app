node {
    stage('Build') {
        docker.image('python:2-alpine').inside{
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
        }
    }
    stage('Test') {
        docker.image('qnib/pytest').inside{
            sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
            junit 'test-reports/results.xml'
        }
    }
    stage("Manual Approval"){
        input message: 'Lanjutkan ke tahap Deploy?', ok: 'Proceed'
    }  
    stage('Deliver') {
        checkout scm
        sh 'docker run --rm -v \$(pwd)/sources:/src cdrx/pyinstaller-linux:python2 "pyinstaller -F add2vals.py"'
        archiveArtifacts 'sources/dist/add2vals'
        sleep time: 10, unit: 'SECONDS'
    } 
}
