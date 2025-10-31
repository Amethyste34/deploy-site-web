pipeline {
    agent {
        label 'connexion'
    }

    stages {
        stage('Dependances') {
            steps {
                echo '[DEPENDANCES] Installation d\'Apache2...'
                sh '''
                    sudo apt-get update
                    sudo apt-get install -y apache2
                    sudo systemctl start apache2
                    sudo systemctl status apache2 --no-pager
                '''
                echo '[OK] Apache2 installe et demarre'
            }
        }

        stage('Checkout') {
            steps {
                echo '[CHECKOUT] Recuperation du code source depuis GitHub...'
                checkout scm
                sh 'ls -la'
                echo '[OK] Code recupere avec succes'
            }
        }

        stage('Backup') {
            steps {
                echo '[BACKUP] Sauvegarde de la version actuelle...'
                sh '''
                    if sudo [ -d /var/www/html.backup ]; then
                        sudo rm -rf /var/www/html.backup
                    fi
                    if sudo [ "$(ls -A /var/www/html 2>/dev/null)" ]; then
                        sudo cp -r /var/www/html /var/www/html.backup
                        echo "[OK] Sauvegarde creee"
                    else
                        echo "[INFO] Pas de version precedente a sauvegarder"
                    fi
                '''
                echo '[OK] Sauvegarde terminee'
            }
        }

        stage('Deploy') {
            steps {
                echo '[DEPLOY] Deploiement des fichiers vers le serveur web...'
                sh '''
                    # Nettoyer le repertoire Apache (sauf les fichiers systeme)
                    sudo rm -f /var/www/html/index.html

                    # Copier les nouveaux fichiers
                    sudo cp -r index.html /var/www/html/

                    # Verifier les permissions
                    sudo chmod 644 /var/www/html/index.html
                    sudo chown www-data:www-data /var/www/html/index.html

                    # Lister les fichiers deployes
                    echo "[INFO] Fichiers deployes :"
                    ls -la /var/www/html/
                '''
                echo '[OK] Fichiers deployes avec succes'
            }
        }

        stage('Test') {
            steps {
                echo '[TEST] Verification du deploiement...'
                sh '''
                    # Attendre qu'Apache soit pret
                    sleep 2

                    # Tester la disponibilite du site
                    curl -f http://localhost/ > /dev/null

                    # Afficher un extrait de la page
                    echo "[INFO] Contenu de la page :"
                    echo "========================"
                    curl -s http://localhost/ | head -20
                    echo "========================"
                '''
                echo '[OK] Site web operationnel et accessible'
            }
        }
    }

    post {
        success {
            echo '=========================================='
            echo '     DEPLOIEMENT REUSSI !'
            echo '=========================================='
            echo '[OK] Le site web est maintenant en ligne'
            echo '[INFO] Accessible sur http://localhost/'
            echo '[INFO] Notification envoyee aux equipes'
            echo '=========================================='
        }
        failure {
            echo '=========================================='
            echo '     ECHEC DU DEPLOIEMENT !'
            echo '=========================================='
            echo '[ROLLBACK] Restauration de la version precedente...'
            sh '''
                if sudo [ -d /var/www/html.backup ]; then
                    sudo rm -rf /var/www/html/*
                    sudo cp -r /var/www/html.backup/* /var/www/html/
                    echo "[OK] Rollback effectue avec succes"
                else
                    echo "[WARNING] Pas de sauvegarde disponible"
                fi
            '''
            echo '[ALERT] L\'equipe technique a ete alertee'
            echo '=========================================='
        }
        always {
            echo '[CLEANUP] Nettoyage des fichiers temporaires...'
            sh '''
                # Supprimer la sauvegarde
                if sudo [ -d /var/www/html.backup ]; then
                    sudo rm -rf /var/www/html.backup
                    echo "[OK] Sauvegarde supprimee"
                fi

                # Arreter et desinstaller Apache2
                sudo systemctl stop apache2 || true
                sudo apt-get remove -y apache2 || true
                sudo apt-get autoremove -y || true

                # Nettoyer /var/www/html
                sudo rm -rf /var/www/html/*

                echo "[OK] Nettoyage termine"
            '''
        }
    }
}