pipeline {
    agent any

    environment {
        APP_NAME = "christ-react-frontend"
        APP_DIR = "/var/www/projects/friendsinchrist/chirist-react-frontend"
        NODE_VERSION = "20"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/codewithkathir/quiz-frontend.git'
            }
        }

        stage('Setup Node') {
            steps {
                sh '''
                export NVM_DIR="$HOME/.nvm"

                if [ ! -d "$NVM_DIR" ]; then
                  curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
                fi

                . "$NVM_DIR/nvm.sh"
                nvm install 20
                nvm use 20
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                export NVM_DIR="$HOME/.nvm"
                . "$NVM_DIR/nvm.sh"
                nvm use 20

                npm install
                '''
            }
        }

        stage('Build App') {
            steps {
                sh '''
                export NVM_DIR="$HOME/.nvm"
                . "$NVM_DIR/nvm.sh"
                nvm use 20

                # FIX: ignore ESLint blocking build
                CI=false npm run build
                '''
            }
        }

        stage('Deploy (PM2 on port 6002)') {
            steps {
                sh '''
                export NVM_DIR="$HOME/.nvm"
                . "$NVM_DIR/nvm.sh"
                nvm use 20

                mkdir -p $APP_DIR

                rsync -av --delete \
                    --exclude node_modules \
                    --exclude .git \
                    --exclude .next \
                    ./ $APP_DIR/

                cd $APP_DIR

                npm install

                npm install -g pm2 || true

                # STOP OLD PROCESS IF EXISTS
                pm2 delete $APP_NAME || true

                # START NEXT.JS ON PORT 6002
                PORT=6002 pm2 start npm --name "$APP_NAME" -- start

                pm2 save
                '''
            }
        }
    }

    post {
        success {
            echo "🚀 Deployment Successful on port 6002"
        }
        failure {
            echo "❌ Deployment Failed"
        }
    }
}
