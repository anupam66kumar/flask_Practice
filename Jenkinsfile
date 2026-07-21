pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Setting up virtual environment and installing dependencies...'
                sh 'python3 -m venv venv'
                sh '. venv/bin/activate && pip install --upgrade pip'
                sh '. venv/bin/activate && pip install -r requirements.txt'
            }
        }
        stage('Test') {
            steps {
                echo 'Running pytest...'
                sh '. venv/bin/activate && pytest'
            }
        }
        stage('Deploy') {
			steps {
				echo 'Deploying application to staging...'
				sh 'pkill -u jenkins -f app.py || true' // Stop any old instance running
				sh 'nohup venv/bin/python app.py > app.log 2>&1 &' // Run app in the background
				echo 'Application deployed and running on port 5000!'
			}
		}
    }
}