pipeline {
    agent none

    stages{
        stage('worker-build'){
            when{
                changeset "**/worker/**"
            }
            agent{
                docker{
                    image 'maven:3.9.9-sapmachine-21'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            steps{
                echo 'building worker app'
                dir('worker'){
                  sh 'mvn compile'
                }
            }
        }
        stage('worker-test'){
            when{
                changeset "**/worker/**"
            }
            agent{
                docker{
                    image 'maven:3.9.9-sapmachine-21'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            steps{
                echo 'running unit tests on worker app'
                dir('worker'){
                  sh 'mvn clean test'
                }
            }
        }
        stage('worker-package'){
            when{
                branch 'master'
                changeset "**/worker/**"
            }
            agent{
                docker{
                    image 'maven:3.9.9-sapmachine-21'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            steps{
                echo 'packaging worker app into a jarfile'
                dir('worker'){
                  sh 'mvn package -DskipTests'
                  archiveArtifacts artifacts: '**/target/*.jar', fingerprint:  true
                }
            }
        }
        stage('worker-docker-package'){
            when{
                branch 'master'
                changeset "**/worker/**"
            }
            agent any
            steps{
                echo 'Packaging worker app with docker'
                script{
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                        def workerImage = docker.build("montemin/worker:v${env.BUILD_ID}", "./worker")
                        workerImage.push()
                        workerImage.push("latest")
                    }
                }
            }
        }

        stage('result-build'){
            when{
                changeset "**/result/**"
            }
            agent{
                docker{
                    image 'node:18-slim'
                    args '-u root'
                }
            }
            steps{
                echo 'building result app'
                dir('result'){
                  sh 'npm install'
                }
            }
        }
        stage('result-test'){
            when{
                changeset "**/result/**"
            }
            agent{
                docker{
                    image 'node:18-slim'
                    args '-u root'
                }
            }
            steps{
                echo 'running unit tests on result app'
                dir('result'){
                  sh 'npm install'
                  sh 'npm test'
                }
            }
        }
        stage('result-docker-package'){
            when{
                branch 'master'
                changeset "**/result/**"
            }
            agent any
            steps{
                echo 'Packaging result app with docker'
                script{
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                        def resultImage = docker.build("montemin/result:v${env.BUILD_ID}", "./result")
                        resultImage.push()
                        resultImage.push("latest")
                    }
                }
            }
        }


         stage('vote-build'){
             when{
                 changeset "**/vote/**"
             }
             agent{
                 docker{
                     image 'python:3.11-slim'
                     args '--user root'
                     }
                     }
             steps{
                 echo 'Compiling vote app.'
                 dir('vote'){

                         sh "pip install -r requirements.txt"

                 }
             }
         }
         stage('vote-test'){
            when{
                changeset "**/vote/**"
            }
             agent {
                 docker{
                     image 'python:3.11-slim'
                     args '--user root'
                     }
                     }
             steps{
                 echo 'Running Unit Tests on vote app.'
                 dir('vote'){

                         sh "pip install -r requirements.txt"
                         sh 'nosetests -v'
                 }
             }
         }

       stage('vote-docker-package'){
           when{
               branch 'master'
               changeset "**/vote/**"
           }
           agent any

           steps{
             echo 'Packaging vote app with docker'
             script{
               docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                   // ./vote is the path to the Dockerfile that Jenkins will find from the Github repo
                   def voteImage = docker.build("montemin/vote:v${env.BUILD_ID}", "./vote")
                   voteImage.push()
                   voteImage.push("${env.BRANCH_NAME}")
                   voteImage.push("latest")
               }
             }
           }
       }

      stage('deploy to dev'){
         agent any
         when{
           branch 'master'
         }
         steps{
           echo 'Deploy instavote app with docker compose'
           sh 'docker-compose up -d'
         }
      }

    }
    post{
        always{
            echo 'the job is complete'
    }
  }
}
