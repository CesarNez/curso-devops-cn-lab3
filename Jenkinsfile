pipeline{
    agent any
    stages{
        stage("Integración continua"){
            agent{
                docker{
                    image "node:24"
                    reuseNode true
                }
            }
            stages{
                stage("1. Instalar dependencias"){
                    steps{                
                        sh "npm install"
                    }
                }
                stage("2. Lint (debug)"){
                    steps{
                        sh "npm run lint"
                    }
                }
                stage("3. Pruebas"){
                    steps{                
                        sh "npm run test"
                    }
                }
                stage("4. Construir aplicación"){
                    steps{
                        sh "npm run build"
                    }
                }
            }
        }
        stage("Generar imagen docker"){
            steps{
                sh "docker build -t curso-devops-lab3 ."
                script{
                    docker.withRegistry("https://index.docker.io/v1/","pass_docker"){
                        sh "docker tag curso-devops-lab3 cesarnez/curso-devops-cn-lab3:latest"
                        sh "docker push cesarnez/curso-devops-cn-lab3:latest"
                    }
                    docker.withRegistry("ghcr.io","pass_gh"){
                        sh "docker tag curso-devops-lab3 cesarnez/curso-devops-cn-lab3:latest"
                        sh "docker push ghcr.io/cesarnez/curso-devops-cn-lab3:latest"
                    }
                }
                
            }
        }      
    }

}