// 1) Zaciągniecie repo
// 2) Usunięcie działającej aplikacji
// 3) Instalacja Mavena
// 4) Build 
// 5) Kontener - utworzyć 
// 6) Kontener - odpalić
// 7) Test (selenium)
// 8) Wypchnąć Artifactory
// 9) Wyczyścić


pipeline {
    agent {
        label 'slave'
    }
    environment {
        //Use Pipeline Utility Steps plugin to read information from pom.xml into env variables
        IMAGE = readMavenPom().getArtifactId()
        VERSION = readMavenPom().getVersion()
        ANSIBLE = tool name: 'Ansible', type: 'com.cloudbees.jenkins.plugins.customtools.CustomTool'
        //IMAGE = "?"
        //VERSION = "?"
    }
    tools {
        terraform 'Terraform'
        maven "m3"
    }
    stages {
        stage('Git pull') {
            steps {
                checkout scm
                //git branch: 'final', url: 'https://github.com/Maclawg/panda_application.git'
            }
        }
        stage('Clean running pandaapp') {
            steps {
                sh "docker rm -f pandaapp || true"
                // sh "mvn clean install"
            }
        }
        stage('Build') {
            steps {
                sh "mvn clean install"
            }
        }
        stage('Container create') {
            steps {
                sh "mvn package -Pdocker"
            }
        }
        stage('Container run') {
            steps {
                sh "echo Run container"
                sh "docker run -dt -p 0.0.0.0:8080:8080 --name pandaapp ${IMAGE}:${VERSION}"
            }
        }
        stage('Run selenium tests') {
            steps {
                sh "mvn test -Pselenium"
            }
        }
        stage('Push to artifactory') {
            steps {
                configFileProvider([configFile(fileId: 'be95abdd-99e9-445b-9b2f-cf9543bd0375', variable: 'MAVEN_GLOBAL_SETTINGS')]) {
                    sh "mvn -gs $MAVEN_GLOBAL_SETTINGS deploy -Dmaven.test.skip=true"
                }
            }
        }
        stage('Run terraform') {
            steps {
                dir('infrastructure/terraform') {
                    sh 'terraform init && terraform apply -var-file ./terraform/panda.tfvars -auto-approve'
                }
            }
        }
        stage('Copy Ansible role') {
            steps {
                sh 'cp -r infrastructure/ansible/panda/ /etc/ansible/roles/'
            }
        }
        stage('Run Ansible') {
            steps {
                dir('infrastructure/ansible') {
                    sh 'chmod 600 ../terraform.pem'
                    sh 'ansible-playbook -i ./inventory playbook.yml'
                }    
            }
        }
    }
    post {
        always {
            sh 'docker stop pandaapp'
            deleteDir()
        }
    }
}


