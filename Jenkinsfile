pipeline {
    agent {
        label 'ridley'
    }

    stages {
        stage("Cleaning workspace") {
            steps {
                cleanWs()
                checkout scm
                echo 'Building $JOB_NAME...'
            }
        }
        stage("Cloning git") {
            steps {
                git branch: "main", url: "https://github.com/Davi-Coelho/traefik.git"
            }
        }
        stage("Setting configuration files") {
            steps {
                script {
                    ACME_FILE = sh (
                        script: 'find . -name acme.json',
                        returnStatus: true
                    )

                    echo "acme: ${ACME_FILE}"

                    if (ACME_FILE == 0) {
                        withCredentials([usernamePassword(credentialsId: 'traefik_user',
                                                          passwordVariable: 'PASSWORD',
                                                          usernameVariable: 'USERNAME')]) {
                            USER_PASSWORD = sh (
                                script: 'htpasswd -nb $USERNAME $PASSWORD',
                                returnStdout: true
                            ).trim()
                        }
                        echo "user_password: ${USER_PASSWORD}"
                        sh "sed 's/your_email/${EMAIL}/' traefik.sample.toml > traefik.toml"
                        sh "sed 's/USER:PASSWORD/${USER_PASSWORD}/' traefik_dynamic.sample.toml > traefik_dynamic.toml"
                        sh "sed -i 's/your_domain/${DOMAIN}/' traefik_dynamic.toml"
                        sh "cp ./acme.sample.json ./acme.json"
                    }
                }
            }
        }
        stage("Stopping containers") {
            steps {
                sh "docker compose down"
            }
        }
        stage("Running containers") {
            steps {
                sh "docker compose up -d --build"
            }
        }
    }
}
