pipeline {
    agent any

    environment {
        PIPENV_VENV_IN_PROJECT = '1'
        PIPENV_NOSPIN = '1'
        FLASK_DEBUG = '0'
    }

    stages {
        stage('Build') {
            steps {
                sh 'python3 -m py_compile app.py tests/test_app.py'
            }
        }
        stage('Dependency Management') {
            steps {
                sh 'pipenv sync --dev'
                sh 'pipenv graph'
                sh 'pipenv requirements --dev > requirements-audit.txt'
            }
        }
        stage('Test') {
            steps {
                sh 'pipenv run pytest -v tests/'
            }
        }
        stage('Analyze') {
            steps {
                sh 'pipenv run flake8 app.py tests --max-line-length=100'
                sh 'pipenv run bandit -q app.py'
                sh 'pipenv run pip-audit --disable-pip --no-deps -r requirements-audit.txt -f json -o dependency-audit.json'
            }
        }
        stage('Deploy') {
            steps {
                sh '''
                    mkdir -p deploy
                    cp app.py deploy/app.py
                    pipenv run python -c "import sys; sys.path.insert(0, 'deploy'); from app import app; r = app.test_client().get('/health'); assert r.status_code == 200 and r.get_json() == {'status': 'ok'}"
                '''
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: 'dependency-audit.json', allowEmptyArchive: true
        }
        success { echo 'Pipeline completado correctamente.' }
        failure { echo 'Pipeline fallido: revisar Console Output y el informe de auditoria.' }
    }
}
