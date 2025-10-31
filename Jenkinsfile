pipeline {
    agent {
        label 'connexion'
    }

    stages {
        stage('Dependances') {
            steps {
                echo '📦 Installation d\'Apache2...'
                sh '''
                    sudo apt-get update
                    sudo apt-get install -y apache2
                    sudo systemctl start apache2
                    sudo systemctl status apache2 --no-pager
                '''
                echo '✅ Apache2 installé et démarré'
            }
        }

        stage('Checkout') {
            steps {
                echo '📥 Récupération du code source depuis GitHub...'
                checkout scm
                sh 'ls -la'
                echo '✅ Code récupéré avec succès'
            }
        }

        stage('Backup') {
            steps {
                echo '💾 Sauvegarde de la version actuelle...'
                sh '''
                    if sudo [ -d /var/www/html.backup ]; then
                        sudo rm -rf /var/www/html.backup
                    fi
                    if sudo [ "$(ls -A /var/www/html 2>/dev/null)" ]; then
                        sudo cp -r /var/www/html /var/www/html.backup
                        echo "✅ Sauvegarde créée"
                    else
                        echo "ℹ️  Pas de version précédente à sauvegarder"
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
                    sudo rm -f /var/www/html/index.html

                    # Copier les nouveaux fichiers
                    sudo cp -r index.html /var/www/html/

                    # Vérifier les permissions
                    sudo chmod 644 /var/www/html/index.html
                    sudo chown www-data:www-data /var/www/html/index.html

                    # Lister les fichiers déployés
                    echo "📂 Fichiers déployés :"
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
                    echo "📄 Contenu de la page :"
                    echo "========================"
                    curl -s http://localhost/ | head -20
                    echo "========================"
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
                if sudo [ -d /var/www/html.backup ]; then
                    sudo rm -rf /var/www/html/*
                    sudo cp -r /var/www/html.backup/* /var/www/html/
                    echo "✅ Rollback effectué avec succès"
                else
                    echo "⚠️  Pas de sauvegarde disponible"
                fi
            '''
            echo '🚨 L\'équipe technique a été alertée'
            echo '❌ =========================================='
        }
        always {
            echo '🧹 Nettoyage des fichiers temporaires...'
            sh '''
                # Supprimer la sauvegarde
                if sudo [ -d /var/www/html.backup ]; then
                    sudo rm -rf /var/www/html.backup
                    echo "✅ Sauvegarde supprimée"
                fi

                # Arrêter et désinstaller Apache2
                sudo systemctl stop apache2 || true
                sudo apt-get remove -y apache2 || true
                sudo apt-get autoremove -y || true

                # Nettoyer /var/www/html
                sudo rm -rf /var/www/html/*

                echo "✅ Nettoyage terminé"
            '''
        }
    }
}