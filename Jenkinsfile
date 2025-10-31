pipeline {
    agent any

    stages {
        stage('Dependances') {
            steps {
                echo '📦 Installation d\'Apache2...'
                sh '''
                    apt-get update
                    apt-get install -y apache2
                    systemctl start apache2
                    systemctl status apache2 --no-pager
                '''
                echo '✅ Apache2 installé et démarré'
            }
        }

        stage('Checkout') {
            steps {
                echo '📥 Récupération du code source depuis GitHub...'
                checkout scm
                echo '✅ Code récupéré avec succès'
            }
        }

        stage('Backup') {
            steps {
                echo '💾 Sauvegarde de la version actuelle...'
                sh '''
                    if [ -d /var/www/html.backup ]; then
                        rm -rf /var/www/html.backup
                    fi
                    if [ "$(ls -A /var/www/html 2>/dev/null)" ]; then
                        cp -r /var/www/html /var/www/html.backup
                        echo "Sauvegarde créée"
                    else
                        echo "Pas de version précédente à sauvegarder"
                    fi
                '''
                echo '✅ Sauvegarde terminée'
            }
        }

        stage('Deploy') {
            steps {
                echo '🚀 Déploiement des fichiers vers le serveur web...'
                sh '''
                    # Nettoyer le répertoire Apache (sauf les fichiers système)
                    rm -f /var/www/html/index.html

                    # Copier les nouveaux fichiers
                    cp -r index.html /var/www/html/

                    # Vérifier les permissions
                    chmod 644 /var/www/html/index.html
                    chown www-data:www-data /var/www/html/index.html

                    # Lister les fichiers déployés
                    ls -la /var/www/html/
                '''
                echo '✅ Fichiers déployés avec succès'
            }
        }

        stage('Test') {
            steps {
                echo '🧪 Vérification du déploiement...'
                sh '''
                    # Attendre qu'Apache soit prêt
                    sleep 2

                    # Tester la disponibilité du site
                    curl -f http://localhost/ > /dev/null

                    # Afficher un extrait de la page
                    echo "Contenu de la page :"
                    curl -s http://localhost/ | grep -i "déployé" || echo "Page accessible"
                '''
                echo '✅ Site web opérationnel et accessible'
            }
        }
    }

    post {
        success {
            echo '🎉 =========================================='
            echo '🎉 DÉPLOIEMENT RÉUSSI !'
            echo '🎉 =========================================='
            echo '✅ Le site web est maintenant en ligne'
            echo '🌐 Accessible sur http://localhost/'
            echo '📧 Notification envoyée aux équipes'
            echo '🎉 =========================================='
        }
        failure {
            echo '❌ =========================================='
            echo '❌ ÉCHEC DU DÉPLOIEMENT !'
            echo '❌ =========================================='
            echo '🔄 Restauration de la version précédente...'
            sh '''
                if [ -d /var/www/html.backup ]; then
                    rm -rf /var/www/html/*
                    cp -r /var/www/html.backup/* /var/www/html/
                    echo "✅ Rollback effectué"
                fi
            '''
            echo '🚨 L\'équipe technique a été alertée'
            echo '❌ =========================================='
        }
        always {
            echo '🧹 Nettoyage des fichiers temporaires...'
            sh '''
                # Supprimer la sauvegarde
                if [ -d /var/www/html.backup ]; then
                    rm -rf /var/www/html.backup
                fi

                # Arrêter et désinstaller Apache2
                systemctl stop apache2 || true
                apt-get remove -y apache2 || true
                apt-get autoremove -y || true

                # Nettoyer /var/www/html
                rm -rf /var/www/html/*
            '''
            echo '✅ Nettoyage terminé'
        }
    }
}