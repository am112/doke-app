pipeline {
    agent any

    environment {
        APP_NAME = 'my-laravel-app'
        DOCKER_COMPOSE_FILE = './compose.dev.yaml'
        COMPOSER_FLAGS = '--no-interaction --prefer-dist --optimize-autoloader'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Prepare Containers') {
            steps {
                configFileProvider([configFile(fileId: 'my-laravel-env', variable: 'ENV_FILE')]) {
                    echo "Copying managed .env file to workspace..."
                    sh 'cp $ENV_FILE .env'
                }
                echo "Building and starting php-fpm container..."
                sh "docker-compose -f $DOCKER_COMPOSE_FILE up -d --build php-fpm"
            }
        }

        stage('Composer Install & Tests') {
            steps {
                echo "Installing PHP dependencies..."
                sh "docker-compose -f $DOCKER_COMPOSE_FILE exec -T php-fpm composer install $COMPOSER_FLAGS"

                echo "Running Laravel tests..."
                sh "docker-compose -f $DOCKER_COMPOSE_FILE exec -T php-fpm php artisan test"
            }
        }

        stage('Build Assets') {
            steps {
                echo "Ensure php-fpm is running for Wayfinder/NPM builds..."
                sh "docker-compose -f $DOCKER_COMPOSE_FILE up -d php-fpm"

                echo "Installing Node.js dependencies..."
                sh "docker-compose -f $DOCKER_COMPOSE_FILE run --rm workspace npm install"

                echo "Building frontend assets..."
                sh "docker-compose -f $DOCKER_COMPOSE_FILE run --rm workspace npm run build"
            }
        }

        stage('Docker Image Build') {
            steps {
                echo "Building production Docker image..."
                sh "docker build -t $APP_NAME -f docker/common/php-fpm/Dockerfile ."
            }
        }
    }

    post {
        always {
            echo "Cleaning up Docker environment..."
            sh "docker-compose -f $DOCKER_COMPOSE_FILE down"
            sh "docker system prune -f"
        }
        success {
            echo "CI pipeline completed successfully!"
        }
        failure {
            echo "CI pipeline failed. Check the logs above."
        }
    }
}
