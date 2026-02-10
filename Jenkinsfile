pipeline {
    agent any

    environment {
        // Il tuo username Docker Hub corretto
        DOCKER_USER = "giacomo12305"
        IMAGE_NAME = "tiger-app"
        // Il link al tuo repo GitHub
        REPO_URL = "github.com/jacklatigre/tiger-app.git"
    }

    stages {
        stage('Build Image') {
            steps {
                echo "Costruisco l'immagine numero ${BUILD_NUMBER}..."
                sh "docker build -t ${DOCKER_USER}/${IMAGE_NAME}:${BUILD_NUMBER} ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                // Utilizziamo le credenziali salvate su Jenkins con ID 'docker-hub-credentials'
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                    // Usiamo \$USER (dalla credenziale) per il login e DOCKER_USER per il push
                    sh "echo \$PASS | docker login -u \$USER --password-stdin"
                    sh "docker push ${DOCKER_USER}/${IMAGE_NAME}:${BUILD_NUMBER}"
                }
            }
        }

        stage('Update Manifest (GitOps)') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-credentials', passwordVariable: 'GIT_PASS', usernameVariable: 'GIT_USER')]) {
                    sh """
                        git config user.email "jenkins@tigerserver.com"
                        git config user.name "Jenkins Bot"

                        # Cambia il tag nel file kustomization.yaml nell'overlay dev
                        # Cerca 'newTag: ...' e lo sostituisce con il numero di build corrente
                        sed -i "s/newTag: .*/newTag: '${BUILD_NUMBER}'/" kustomize/overlays/dev/kustomization.yaml

                        git add kustomize/overlays/dev/kustomization.yaml
                        git commit -m "chore: update image tag to ${BUILD_NUMBER} [skip ci]"
                        git push https://${GIT_USER}:${GIT_PASS}@${REPO_URL} HEAD:master
                    """
                }
            }
        }
    }
}
