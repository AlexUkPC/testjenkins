pipeline {
    agent {
        label 'ssh'  
    }
    parameters {
        string(name: 'REF', defaultValue: '\${ghprbActualCommit}', description: 'Commit to build')
    }
    stages {
        stage('Bundle Install') {
            steps {
                sh '/usr/local/bin/docker-compose run --rm web_testjenkins bundle install'
            }
        }
        stage('Webpacker Install') {
            steps {
                sh '/usr/local/bin/docker-compose run --rm web_testjenkins gem install nokogiri --platform=ruby; bin/bundle config set force_ruby_platform true; bin/rails webpacker:install'
            }
        }
        stage('Stop old containers') {
            steps {
                sh '/usr/local/bin/docker-compose stop'
            }
        }
        stage('Start server') {
            steps {
                sh '/usr/local/bin/docker-compose up -d --remove-orphans'
            }
        }
        stage('Create database') {
            steps {
                sh '/usr/local/bin/docker-compose exec -T --user "$(id -u):$(id -g)" web_testjenkins bin/rails db:create'
            }
        }
        stage('Migrate database') {
            steps {
                sh '/usr/local/bin/docker-compose exec -T --user "$(id -u):$(id -g)" web_testjenkins bin/rails db:migrate'
            }
        }
        stage('Wait for server to start') {
            steps {
                timeout(10) {
                    waitUntil {
                        script {
                            try {
                                def response = httpRequest 'http://0.0.0.0:3900'
                                return (response.status == 200)
                            }
                            catch (exception) {
                                return false
                            }
                        }
                    }
                }
            }
        }
        stage('Unit test') {
            steps {
                sh '/usr/local/bin/docker-compose exec -T --user "$(id -u):$(id -g)" web_testjenkins bin/rails test:models'
            }   
        } 
    }
}