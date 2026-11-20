pipeline {
    agent any

    environment {
        DEPLOY_DIR = '/tmp/flask-deployment'
        APP_PORT = '5001'
    }

    stages {

        stage('Build') {
            steps {
                echo 'Building Flask application...'
                sh '''
                    python3 -m venv venv
                    venv/bin/pip install -r requirements.txt
                '''
            }
        }

        stage('Test') {
            steps {
                echo 'Running unit tests...'
                sh '''
                    venv/bin/python -m unittest test_app.py
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying application...'
                sh '''
                    rm -rf "$DEPLOY_DIR"
                    mkdir -p "$DEPLOY_DIR"
                    cp app.py requirements.txt "$DEPLOY_DIR/"
                '''
            }
        }

        stage('Run Application') {
            steps {
                echo 'Starting deployed Flask application...'
                sh '''
                    cd "$DEPLOY_DIR"

                    APP_PORT="$APP_PORT" nohup "$WORKSPACE/venv/bin/python" app.py \
                    > application.log 2>&1 &

                    echo $! > application.pid
                '''
            }
        }

        stage('Test Application') {
            steps {
                echo 'Verifying deployed application...'
                sh '''
                    python3 -c "
import time
import urllib.request

url = 'http://127.0.0.1:5001/'

for attempt in range(15):
    try:
        response = urllib.request.urlopen(url, timeout=2)
        body = response.read().decode()

        assert response.status == 200
        assert 'Flask CI/CD Pipeline is working!' in body

        print('Post-deployment verification passed!')
        break

    except Exception:
        time.sleep(1)

else:
    raise Exception('Deployed application verification failed')
"
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline completed successfully!'
        }

        failure {
            echo 'CI/CD Pipeline FAILED!'
        }

        always {
            echo 'Pipeline execution finished.'
        }
    }
}