pipeline {
    agent any

    options {
        timeout(time: 15, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
        disableConcurrentBuilds()
    }

    parameters {
        choice(name: 'ENVIRONMENT', choices: ['dev', 'test', 'prod'], description: 'Environnement cible')
        string(name: 'VERSION', defaultValue: '1.0.0', description: 'Version de artefact (semver)')
        booleanParam(name: 'DRY_RUN', defaultValue: true, description: 'Exécution à blanc')
        string(name: 'CHANGE_REFERENCE', defaultValue: '', description: 'Référence du ticket de changement')
    }

    stages {
        stage('Préparation') {
            steps {
                echo "Préparation du build #${env.BUILD_NUMBER}"
                sh 'git rev-parse --short HEAD'
                sh 'mkdir -p artifacts'
            }
        }

        stage('Validation') {
            steps {
                script {
                    def allowedEnvs = ['dev', 'test', 'prod']
                    if (!allowedEnvs.contains(params.ENVIRONMENT)) {
                        error("ENVIRONMENT invalide : ${params.ENVIRONMENT}. Valeurs autorisées : ${allowedEnvs}")
                    }
                    if (!(params.VERSION ==~ /^\d+\.\d+\.\d+$/)) {
                        error("VERSION invalide : ${params.VERSION}. Format attendu X.Y.Z")
                    }
                    if (params.CHANGE_REFERENCE.trim() == '') {
                        error("CHANGE_REFERENCE est obligatoire")
                    }
                }
                echo "Contexte validé : env=${params.ENVIRONMENT}, version=${params.VERSION}, dry_run=${params.DRY_RUN}"
            }
        }

        stage('Exécution') {
            steps {
                sh '''
                    SHORT_SHA=$(git rev-parse --short HEAD)
                    printf 'build=%s\ncommit=%s\nenvironment=%s\nversion=%s\n' \
                        "$BUILD_NUMBER" "$SHORT_SHA" "$ENVIRONMENT" "$VERSION" > artifacts/build-info.txt
                    cat artifacts/build-info.txt
                '''
                script {
                    if (params.DRY_RUN) {
                        echo "DRY_RUN actif : simulation uniquement, aucune action réelle."
                    } else {
                        echo "Exécution réelle sur ${params.ENVIRONMENT} en cours..."
                    }
                }
            }
        }

        stage('Post-traitement') {
            steps {
                archiveArtifacts artifacts: 'artifacts/build-info.txt', fingerprint: true
                echo "Artefact archivé pour le build #${env.BUILD_NUMBER}"
            }
        }
    }

    post {
        success {
            echo "Build #${env.BUILD_NUMBER} réussi — ${params.ENVIRONMENT} / v${params.VERSION} / ${params.CHANGE_REFERENCE}"
        }
        failure {
            echo "Build #${env.BUILD_NUMBER} en échec — vérifier les paramètres fournis."
        }
        always {
            echo "Fin du pipeline — statut final : ${currentBuild.currentResult}"
        }
    }
}
