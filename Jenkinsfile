pipeline{

    agent none 


    stages{
        stage("worker build"){
            agent{
                 docker{
                   image 'maven:3.6.1-jdk-8-alpine'
                   args '-v $HOME/.m2/root/.m2'
                 }
            } 

            when{
              changeset "**/worker/**"
            }
            steps{
                echo "Compiling worker app..."
                dir("worker"){
                  sh 'mvn compile'
                }
            }
        }
        stage("worker test"){
            agent{
                 docker{
                   image 'maven:3.6.1-jdk-8-alpine'
                   args '-v $HOME/.m2/root/.m2'
                 }
            }
            when{
              changeset "**/worker/**"
            }
            steps{
                echo "Running Unit Tests on worker app..."
                dir("worker"){
                  sh 'mvn clean test'
                }
            }
        }
        stage("worker package"){
            agent{
                 docker{
                   image 'maven:3.6.1-jdk-8-alpine'
                   args '-v $HOME/.m2/root/.m2'
                 }
            }
            when{
              changeset "**/worker/**"
              branch 'master'
            }
            steps{
                echo "Packaging worker app..."
                dir("worker"){
                  sh 'mvn package -DskipTests'
                  archiveArtifacts artifacts: '**/target/*.jar'
                }
            }
        }

       stage('worker docker-package'){
         agent any
         when{
            changeset "**/worker/**"
	    branch 'master'
         }
         steps{
           echo 'Packaging worker app with docker'
           script{
             docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                def workerImage = docker.build("aydakca/worker:v${env.BUILD_ID}", "./worker")
                workerImage.push()
                workerImage.push("${env.BRANCH_NAME}")
             }
          } 
      }

    stages{
        stage("result build"){
            agent {
              docker{
                image 'node:8.16-alpine'
              }
            }
            when{
              changeset "**/result/**"
            }
            steps{
                echo "Building result app..."
                dir("result"){
                  sh 'npm install'
                }
            }
        }
        stage("result test"){
            agent {
              docker{
                image 'node:8.16-alpine'
              }
            }
            when{
              changeset "**/result/**"
            }
            steps{
                echo "Testing result app..."
                dir("result"){
                  sh 'npm install'
		  sh 'npm test'
                }
            }
        }
        stage('result docker-package'){
          agent any
          when{
             changeset "**/result/**"
	     branch 'master'
          }
          steps{
            echo 'Packaging result app with docker'
            script{
              docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                 def resultImage = docker.build("aydakca/result:v${env.BUILD_ID}", "./result")
                 resultImage.push()
                 resultImage.push("latest")
              }
           }
       }
    }


stage('vote build'){
          agent{
            docker{
              image 'python:2.7.16-slim'
              args '--user root'
            }
          }
          when{
            changeset "**/vote/**"
          }
          steps{
            echo 'Compiling vote app'
            dir('vote'){
              sh 'pip install -r requirements.txt'
            }
          }

      }
      stage('vote test'){
          agent{
            docker{
              image 'python:2.7.16-slim'
              args '--user root'
            }
          }
          when{
            changeset "**/vote/**"
          }
          steps{
            echo 'Running Unit Tests on vote app'
            dir('vote'){
              sh 'pip install -r requirements.txt'
              sh 'nosetests -v'
            }
          }
      }
      stage('vote docker-package'){
          agent any
          when{
            changeset "**/vote/**"
          }
          steps{
            echo 'Packaging vote app with docker'
            script{
              docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                  def voteImage = docker.build("initcron/vote:v-adb-${env.BUILD_ID}", "./vote")
                  voteImage.push()
                  voteImage.push("latest")
              }
            }
          }
      }
}

    }
    
    post{
        always{
            echo "This pipeline run is completed.."
        }
	failure{
	    echo "Failure"
	}
	success{
	    echo "Success"
	}
    }
}
