pipeline {
    agent any

    stages {
        stage('PullCode') {
            steps {
                echo 'Pull Code..'
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'AWel11', url: 'https://github.com/AWel11/EnglishPal.git']]])
            }
        }
        stage('MakeDatabasefile') {
	    steps {
	        sh 'touch ./app/static/wordfreqapp.db && rm -f ./app/static/wordfreqapp.db' 
	        sh 'cat ./app/static/wordfreqapp.sql | sqlite3 ./app/static/wordfreqapp.db'
	    }
	}
        stage('BuildIt') {
            steps {
                echo 'Building..'
		sh 'sudo docker build -t englishpal_lw .'
		sh 'sudo docker stop $(docker ps -aq)'
		sh 'sudo docker run -d -p 6000:80 -v /var/lib/jenkins/workspace/EnglishPal_Pipeline_master/app/static/frequency:./app/static/frequency -t englishpal_lw'
            }
        }
        stage('TestIt') {
            steps {
                echo 'Testing..'
		sh 'sudo docker run -d -p 4500:4444 selenium/standalone-chrome'
		sh 'pip3 install pytest -U -q'
		sh 'pip3 install selenium -U -q'
		sh 'python3 -m pytest -v -s ./app/test'
            }
        }
        stage('DeployIt') {
            steps {
                echo 'Deploying....'
            }
        }
    }
}
