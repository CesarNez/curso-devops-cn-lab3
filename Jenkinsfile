def tagAndPush(String localImage, String repo, String registry, String credential) {

    docker.withRegistry(registry, credential) {
        sh "docker tag ${localImage} ${repo}:latest"
        sh "docker tag ${localImage} ${repo}:${env.BUILD_NUMBER}"
        sh "docker tag ${localImage} ${repo}:${env.APP_SEMANTIC_VERSION}"
        sh "docker push ${repo}:latest"
        sh "docker push ${repo}:${env.BUILD_NUMBER}"
        sh "docker push ${repo}:${env.APP_SEMANTIC_VERSION}"
    }
}

pipeline{
    agent any
    environment {
        IMAGE_NAME = "curso-devops-cn-lab3"
        DH_REPO    = "cesarnez/curso-devops-cn-lab3"
        GHCR_REPO  = "ghcr.io/cesarnez/curso-devops-cn-lab3"
        K8S_NAMESPACE  = "cnunez"
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
                stage("Versionamiento") {
                    steps {
                        script {
                            env.APP_SEMANTIC_VERSION = sh(
                                script: 'npm pkg get version | tr -d \'"\'',
                                returnStdout: true
                            ).trim()
                            echo "Version semantica detectada: ${env.APP_SEMANTIC_VERSION}"
                        }
                    }
                }
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
                sh "docker build -t curso-devops-cn-lab3 ."
                script {
                    if (!env.APP_SEMANTIC_VERSION?.trim()) {
                        error("APP_SEMANTIC_VERSION no definida en el stage anterior")
                    }
                    tagAndPush(env.IMAGE_NAME, env.DH_REPO, "https://index.docker.io/v1/", "pass_docker")
                    tagAndPush(env.IMAGE_NAME, env.GHCR_REPO, "https://ghcr.io", "pass_gh")
                }
            }
        }
        stage("4. Despliegue continuo en ambiente"){
            agent {
                docker {
                    image 'alpine/k8s:1.34.6'
                    reuseNode true
                }
            }
            steps{
                script {
                    if (!env.APP_SEMANTIC_VERSION?.trim()) {
                        error("APP_SEMANTIC_VERSION no definida para el despliegue")
                    }
                }
                withKubeConfig([credentialsId: 'pass-k8']) {
                    sh """
                        kubectl -n ${env.K8S_NAMESPACE} set image deployment/${env.K8S_DEPLOYMENT} ${env.K8S_CONTAINER}=${env.DH_REPO}:${env.APP_SEMANTIC_VERSION}
                        kubectl -n ${env.K8S_NAMESPACE} rollout status deployment/${env.K8S_DEPLOYMENT}
                    """
                }
            }
        }
    }
}