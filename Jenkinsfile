pipeline {
    agent none

    options {
        timeout(time: 15, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    parameters {
        choice(name: 'ENVIRONMENT', choices: ['dev', 'test', 'prod'], description: 'Environnement cible')
        string(name: 'VERSION', defaultValue: '1.0.0', description: 'Version de artefact')
        booleanParam(name: 'DRY_RUN', defaultValue: true, description: 'Exécution à blanc')
        string(name: 'CHANGE_REFERENCE', defaultValue: '', description: 'Référence du ticket')
    }

    stages {
        stage('Validation') {
            agent any
            steps {
                script {
                    def allowedEnvs = ['dev', 'test', 'prod']
                    if (!allowedEnvs.contains(params.ENVIRONMENT)) {
                        error("ENVIRONMENT invalide : ${params.ENVIRONMENT}")
                    }
                    if (!(params.VERSION ==~ /^\d+\.\d+\.\d+$/)) {
                        error("VERSION invalide : ${params.VERSION}")
                    }
                    if (params.CHANGE_REFERENCE.trim() == '') {
                        error("CHANGE_REFERENCE est obligatoire")
                    }
                }
                echo "Contexte validé : env=${params.ENVIRONMENT}, version=${params.VERSION}"
            }
        }

        stage('Identité AWS') {
            agent { label 'aws-lab' }
            steps {
                echo "Exécution sur l'agent étiqueté aws-lab"
                sh 'aws sts get-caller-identity'
            }
        }

        stage('Inventaire S3') {
            agent { label 'aws-lab' }
            steps {
                sh "aws s3api list-buckets --query 'Buckets[].Name' --output table"
            }
        }

        stage('Inventaire EC2') {
            agent { label 'aws-lab' }
            steps {
                sh "aws ec2 describe-instances --query 'Reservations[].Instances[].InstanceId' --output table"
            }
        }

        stage('Déclenchement job aval') {
            agent any
            steps {
                build job: 'TP4-Job-Aval', parameters: [
                    string(name: 'ENVIRONMENT', value: params.ENVIRONMENT),
                    string(name: 'CHANGE_REFERENCE', value: params.CHANGE_REFERENCE)
                ]
            }
        }
    }

    post {
        always {
            echo "Build #${env.BUILD_NUMBER} terminé — statut : ${currentBuild.currentResult}"
        }
    }
}
