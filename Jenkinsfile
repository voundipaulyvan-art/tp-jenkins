pipeline {

    agent any

    tools {
        maven 'Maven3'
        jdk   'JDK21'
    }

    parameters {
        choice(
            name:        'ENVIRONMENT',
            choices:     ['dev', 'staging', 'prod'],
            description: 'Environnement cible'
        )
        booleanParam(
            name:         'SKIP_TESTS',
            defaultValue: false,
            description:  'Passer les tests (urgence uniquement)'
        )
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                echo "Commit : ${env.GIT_COMMIT}"
                echo "Environnement : ${params.ENVIRONMENT}"
            }
        }

        stage('Build') {
            steps {
                bat 'mvn clean compile -B'
            }
        }

        stage('Tests unitaires') {
            when {
                not { expression { return params.SKIP_TESTS } }
            }
            steps {
                bat 'mvn test -B'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
                failure {
                    echo 'Tests unitaires en echec'
                }
            }
        }

        stage('Tests integration') {
            when {
                not { expression { return params.SKIP_TESTS } }
            }
            steps {
                bat 'mvn verify -Dsurefire.skip=true -B'
            }
            post {
                always {
                    junit '**/target/failsafe-reports/*.xml'
                }
            }
        }

        stage('Couverture JaCoCo') {
            steps {
                bat 'mvn jacoco:report -B'
            }
            post {
                always {
                    jacoco(
                        execPattern:         '**/target/jacoco.exec',
                        classPattern:        '**/target/classes',
                        sourcePattern:       '**/src/main/java',
                        minimumLineCoverage: '70'
                    )
                }
            }
        }

        stage('Qualite') {
            steps {
                bat 'mvn checkstyle:checkstyle pmd:pmd pmd:cpd spotbugs:spotbugs -B'
            }
            post {
                always {
                    echo 'Rapports qualite generes dans target/'
                }
            }
        }

        stage('Archive') {
            steps {
                archiveArtifacts(
                    artifacts:         '**/target/*.jar',
                    fingerprint:       true,
                    allowEmptyArchive: false
                )
            }
        }

        stage('Validation PROD') {
            when {
                expression { return params.ENVIRONMENT == 'prod' }
            }
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    input(
                        message:   'Deployer en PRODUCTION ?',
                        ok:        'Oui, deployer',
                        submitter: 'admin'
                    )
                }
            }
        }

    }

    post {
        always {
            echo "Pipeline terminee - statut : ${currentBuild.currentResult}"
        }
        failure {
            echo "Build en echec - ${env.JOB_NAME} #${env.BUILD_NUMBER}"
        }
        fixed {
            echo "Build de nouveau stable - ${env.JOB_NAME} #${env.BUILD_NUMBER}"
        }
    }

}