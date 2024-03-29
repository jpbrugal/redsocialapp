pipeline {
    agent any

    environment {
        REPO_URL = "https://github.com/jpbrugal/redsocialapp"
    }

    stages {
        stage('Desarrollo') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'jenkinsGitHubToken', variable: 'GIT_TOKEN')]) {
                        withCredentials([file(credentialsId: 'firebaseCredentials', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]){
                            retryOrAbort('Desarrollo', {
                            desarrolloActions()
                            developmentTests()
                        })

                        }
                    }
                }
            }
        }

        stage('QA') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'jenkinsGitHubToken', variable: 'GIT_TOKEN')]) {
                        withCredentials([file(credentialsId: 'firebaseCredentials', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]){
                            
                            retryOrAbort('QA', {
                            qaActions()
                            qaTests()
                        })

                        }

                        
                    }
                }
            }
        }

        stage('Producción') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'jenkinsGitHubToken', variable: 'GIT_TOKEN')]) {
                        withCredentials([file(credentialsId: 'firebaseCredentials', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]){
                            
                            retryOrAbort('Produccion', {
                            produccionActions()
                            produccionTests()
                            })

                        }
                        
                    }
                }
            }
        }

    //     stage('Deploy to Firebase') {
    //     when {
    //         expression { currentBuild.resultIsBetterOrEqualTo('SUCCESS') }
    //     }
    //     steps {
    //         script {
    //             withCredentials([file(credentialsId: 'firebaseCredentials', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
    //                 retryOrAbort('Deploy to Firebase', {
    //                     firebaseActions()
    //                 })
    //             }
    //         }
    //     }
    // }
    }

    post {
        always {
            sh 'git checkout master'

            emailext (
                subject: "Pipeline Status: ${BUILD_NUMBER}",
                body: '''<html>
                            <body>
                                <p>Build Status: ${BUILD_STATUS}</p>
                                <p>Build Number: ${BUILD_NUMBER}</p>
                                <p>Check the <a href="${BUILD_URL}">console output</a>.</p>
                            </body>
                        </html>''',
                to: 'jpbrugal81@gmail.com',
                from: 'jenkins@example.com',
                replyTo: 'jenkins@example.com',
                mimeType: 'text/html'    
            )
        }
    }
}

def retryOrAbort(stageName, Closure block) {
    try {
        block.call()
    } catch (Exception e) {
        echo "Error in ${stageName} stage: ${e.message}"
        def userChoice = input(
            id: 'userChoice', message: "Error encountered in ${stageName}. Choose an action:", 
            parameters: [choice(name: 'ACTION', choices: ['retry'], description: 'Select Action')]
        )
        if (userChoice == 'retry') {
            echo "Retrying the ${stageName} stage..."
            block.call()
        } else {
            error "Pipeline aborted by the user in ${stageName} stage."
        }
    }
}

def desarrolloActions() {
    sh 'git config --global credential.helper "!f() { echo username=jpbrugal; echo password=$GIT_TOKEN; }; f"'
    sh 'git checkout desarrollo'
    sh 'git fetch origin master:master'
    sh 'git merge master'
    sh 'git push origin desarrollo'
    sh 'git config --global --unset credential.helper'
}

def developmentTests() {
    sh 'npm install'
    //sh 'npm test'
    sh 'npm run build'
    input(id: 'ApproveDeploy', message: 'Deploy?', ok: 'Yes', submitter: 'jpbrugal,desarollador')
    sh 'firebase deploy --only hosting:desarrollo --token "$FIREBASE_TOKEN"'
    input(id: 'ProceedToQA', message: 'Apbrobar ir a QA?', ok: 'Yes', submitter: 'jpbrugal')
}

def qaActions() {
    sh 'git config --global credential.helper "!f() { echo username=jpbrugal; echo password=$GIT_TOKEN; }; f"'
    sh 'git checkout qa'
    sh 'git fetch origin desarrollo:desarrollo'
    sh 'git merge desarrollo'
    sh 'git push origin qa'
    sh 'git config --global --unset credential.helper'
}

def qaTests() {
    sh 'npx eslint'
    sh 'npx jest'
    input(id: 'ApproveDeploy', message: 'Deploy?', ok: 'Yes', submitter: 'jpbrugal,qa')
    sh 'firebase deploy --only hosting:qa --token "$FIREBASE_TOKEN"'
    input(id: 'ProceedToProduccion', message: 'Apbrobar ir a Produccion?', ok: 'Yes', submitter: 'jpbrugal')
}

def produccionActions() {
    sh 'git config --global credential.helper "!f() { echo username=jpbrugal; echo password=$GIT_TOKEN; }; f"'
    sh 'git checkout produccion'
    sh 'git fetch origin qa:qa'
    sh 'git merge qa'
    sh 'git push origin produccion'
    sh 'git config --global --unset credential.helper'
}

def produccionTests() {
    sh 'npm run test-file -- Login.test.js'
    input(id: 'ApproveDeploy', message: 'Deploy?', ok: 'Yes', submitter: 'jpbrugal')
    sh 'firebase deploy --only hosting:production --token "$FIREBASE_TOKEN"'
   //input(id: 'ProceedToFirebase', message: 'Apbrobar ir a Firebase?', ok: 'Yes')
}

def firebaseActions(){
    withCredentials([file(credentialsId: 'firebaseCredentials', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
        //sh 'firebase deploy --only hosting --token "$FIREBASE_TOKEN"'
        //sh 'firebase deploy --only hosting:production --token "$FIREBASE_TOKEN"'
    }

}
