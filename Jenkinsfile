pipeline {

  // Le pipeline s'exécute sur le Jenkins lui-même
  agent any

  options {
    disableConcurrentBuilds()           
    timeout(time: 45, unit: 'MINUTES') 
  }

  environment {
    DOCKERHUB_USER = 'kane2404'              // compte Docker Hub 
    TAG            = "${env.BUILD_NUMBER}"   // version des images = numéro du build Jenkins
  }

  stages {

    //1 Checkout : récupère le code depuis le dépôt Git du job 
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    //2 Build : construit les 3 images à partir des Dockerfile 
    stage('Build') {
      steps {
        sh '''
          docker build -t bijouterie-frontend:$TAG frontend/
          docker build -t bijouterie-backend:$TAG  backend/
          docker build -t bijouterie-mysql:$TAG    database/
        '''
      }
    }

    // 3. Tag : double tag, le numéro de build (utilisé dans les manifests) et latest (simple commodité sur Docker Hub)
    stage('Tag') {
      steps {
        sh '''
          for s in frontend backend mysql; do
            docker tag bijouterie-$s:$TAG $DOCKERHUB_USER/bijouterie-$s:$TAG
            docker tag bijouterie-$s:$TAG $DOCKERHUB_USER/bijouterie-$s:latest
          done
        '''
      }
    }

    // --- 4. Push : envoie les images vers Docker Hub 
    // Les identifiants viennent du credential Jenkins "dockerhub-credentials".
    // withCredentials masque le token dans les logs ; les guillemets simples '''
    // laissent le SHELL lire les variables (Groovy ne les interpole donc pas).
    // --password-stdin évite de mettre le token dans la ligne de commande.
    stage('Push') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials',
                                          usernameVariable: 'DH_USER',
                                          passwordVariable: 'DH_TOKEN')]) {
          sh '''
            echo "$DH_TOKEN" | docker login -u "$DH_USER" --password-stdin
            for s in frontend backend mysql; do
              docker push $DOCKERHUB_USER/bijouterie-$s:$TAG
              docker push $DOCKERHUB_USER/bijouterie-$s:latest
            done
          '''
        }
      }
    }
  }

  post {
    success {
      echo "Images publiées : ${env.DOCKERHUB_USER}/bijouterie-{frontend,backend,mysql}:${env.TAG}"
    }
    always {
      sh 'docker logout || true'   // ne laisse pas la session Docker Hub ouverte
    }
  }
}
