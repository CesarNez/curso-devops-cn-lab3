pipeline{
    agent any
    environment {
        IMAGE_NAME = "curso-devops-lab3"
        DH_REPO    = "cesarnez/curso-devops-cn-lab3"
        GHCR_REPO  = "ghcr.io/cesarnez/curso-devops-cn-lab3"
        K8S_NAMESPACE  = "curso-lab3"
        K8S_DEPLOYMENT = "curso-lab3-deployment"
        K8S_CONTAINER  = "contenedor-curso-lab3"
    }
    stages{
        stage("1. Integración continua"){
            agent{
                docker{
                    image "node:24"
                    reuseNode true
                }
            }
            stages{
                stage("Instalar dependencias"){
                    steps{                
                        sh "npm install"
                    }
                }
                stage("Lint (debug)"){
                    steps{
                        sh "npm run lint"
                    }
                }
                stage("Pruebas locales"){
                    steps{                
                        sh "npm run test:cov"
                    }
                }
                stage("Construir aplicación"){
                    steps{
                        sh "npm run build"
                    }
                }
            }
        }
        stage("2. QA"){
            agent {
                docker {
                    image 'sonarsource/sonar-scanner-cli'
                    args '--network=devops-infra_default'
                    reuseNode true
                }
            }
            stages{
                stage("Validacion de codigo"){
                    steps{
                        withSonarQubeEnv('sonarqube'){
                            sh 'sonar-scanner'
                        }
                    }
                }
                stage('Validacion quality gate'){
                    steps{
                        script{
                            def qualityGate = waitForQualityGate()
                            if(qualityGate.status != 'OK'){
                                error "La puerta de calidad ha fallado: ${qualityGate.status}"
                            }
                        }
                    }
                }
            }
        }
        stage("3. Generar imagen docker"){
            steps{
                sh "docker build -t curso-devops-lab3 ."
                script{
                    docker.withRegistry("https://index.docker.io/v1/","pass_docker"){
                        sh "docker tag curso-devops-lab3 cesarnez/curso-devops-lab3:latest"
                        sh "docker push cesarnez/curso-devops-lab3:latest"
                    }
                    docker.withRegistry("https://ghcr.io","pass_gh"){
                        sh "docker tag curso-devops-lab3 cesarnez/curso-devops-lab3:latest"
                        sh "docker push ghcr.io/cesarnez/curso-devops-lab3:latest"
                    }
                }
                
            }
        }      
    }

}