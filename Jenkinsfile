// node {
//     stage('Build') {
//         docker.image('python:2-alpine').inside{
//             sh 'python -m py_compile sources/add2vals.py sources/calc.py'
//         }
//     }
//     stage('Test') {
//         docker.image('qnib/pytest').inside{
//             sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
//             junit 'test-reports/results.xml'
//         }
//     }
//     stage('Deliver') {
//         docker.image('cdrx/pyinstaller-linux:python2').inside{
//             sh 'pyinstaller --onefile sources/add2vals.py'
//         }
//         archiveArtifacts 'dist/add2vals'
//             sleep(60)
//     }   
// }

node {
    stage("Build"){
    checkout scm
        docker.image('python:2-alpine').inside {
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
        }
    }

    stage("Test"){
    checkout scm
        try {
            docker.image('qnib/pytest').inside {
                sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
            }
        } finally {
            junit 'test-reports/results.xml'
        }
    }

    if (currentBuild.currentResult == 'SUCCESS') {
        stage("Manual Approval"){
            input message: 'Lanjutkan ke tahap Deploy?', ok: 'Proceed'
        }
        stage("Deploy"){
            checkout scm
            sh 'docker run --rm -v \$(pwd)/sources:/src cdrx/pyinstaller-linux:python2 "pyinstaller -F add2vals.py"'
            archiveArtifacts 'sources/dist/add2vals'

            echo "Deploying to production"
            withCredentials([sshUserPrivateKey(credentialsId: 'jenkins-ec2-ssh', keyFileVariable: 'KEY', usernameVariable: 'USERNAME'), string(credentialsId: 'ec2-host', variable: 'HOST')]) {
                sh 'scp -i $KEY sources/dist/add2vals $USERNAME@$HOST:/home/$USERNAME/dist/add2vals-$(date \'+%F-%H-%M-%S\')'
            }
            echo "Deploy finihed"

            // sleep 10 seconds
            sleep time: 10, unit: 'SECONDS'
        }
    }

}
