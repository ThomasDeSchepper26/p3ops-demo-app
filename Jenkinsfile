pipeline {
    agent any
    
    environment {
        CONTAINER_NAME = 'dotnet6-container'
        PROJECT_PATH = 'src/Server/Server.csproj'
        TEST_PATH = 'tests/Domain.Tests/Domain.Tests.csproj'
        PUBLISH_PATH = 'publish'
    }
    
    stages {
        
        stage('Delete repo') {
            steps {
                sh 'if [ -d p3ops-demo-app ]; then rm -rf p3ops-demo-app; fi'
            }
        }           
        
        stage('Clone repo') {
            steps {
                sh 'git clone --branch main https://github.com/ThomasDeSchepper26/p3ops-demo-app.git'                
            }
        }
       
        stage('Copy files to dotnet6-container') {
            steps {
                script {
                    sh 'docker cp p3ops-demo-app dotnet6-container:'
                }
            }
        }
        
        stage('Restore Dependencies') {
            steps {
                echo 'Restoring .NET dependencies...'
                sh 'docker exec ${CONTAINER_NAME} bash -c "dotnet restore ${PROJECT_PATH}"'
                sh 'docker exec ${CONTAINER_NAME} bash -c "dotnet restore ${TEST_PATH}"'
            }
        }

        stage('Lint Project') {
            steps {
                echo 'Linting .NET project...'
                sh 'docker exec ${CONTAINER_NAME} bash -c "dotnet tool install -g dotnet-format || true"'
                sh 'docker exec ${CONTAINER_NAME} bash -c "~/.dotnet/tools/dotnet-format ${PROJECT_PATH}"'
            }
        }

        stage('Build Project') {
            steps {
                echo 'Building .NET project...'
                sh 'docker exec ${CONTAINER_NAME} bash -c "dotnet build ${PROJECT_PATH} -c Release -o /app/build"'
            }
        }
        
        stage('Test Project') {
            steps {
                echo 'Testing .NET project...'
                sh 'docker exec ${CONTAINER_NAME} bash -c "dotnet test ${TEST_PATH}"'
            }
        }

        stage('Publish Application') {
            steps {
                echo 'Publishing .NET application...'
                sh 'docker exec ${CONTAINER_NAME} bash -c "dotnet publish ${PROJECT_PATH} -c Release -o ${PUBLISH_PATH}"'
            }
        }

        stage('Run Application') {
            steps {
                echo 'Running the .NET application...'
                sh 'docker exec ${CONTAINER_NAME} bash -c "cd ${PUBLISH_PATH} && nohup dotnet Server.dll > /dev/null 2>&1 &"'
            }
        }
    }

    post {
        always {
            echo 'Cleaning up...'
            sh 'docker logout || true'
        }
     }
}
