# Jenkins et CI/CD Alternatifs

> [!info] CI/CD — Intégration et Déploiement Continus
> L'intégration continue (CI) automatise la vérification du code à chaque commit. Le déploiement continu (CD) automatise la livraison jusqu'en production. Ces pratiques réduisent les risques et accélèrent la cadence de livraison.

## Table des matières
1. [[#Comparatif des plateformes CI/CD]]
2. [[#Jenkins — Architecture et installation]]
3. [[#Jenkinsfile — Declarative Pipeline]]
4. [[#Jenkins Shared Libraries]]
5. [[#Plugins Jenkins essentiels]]
6. [[#GitLab CI]]
7. [[#CircleCI]]
8. [[#Patterns CI/CD]]
9. [[#Self-hosted runners]]
10. [[#Sécurité CI/CD]]
11. [[#Exercices Pratiques]]

---

## Comparatif des plateformes CI/CD

| Critère | Jenkins | GitHub Actions | GitLab CI | CircleCI | Travis CI |
|---------|---------|----------------|-----------|----------|-----------|
| **Hébergement** | Self-hosted | Cloud + Self | Cloud + Self | Cloud + Self | Cloud |
| **Pricing** | Gratuit (infra) | Gratuit 2000 min/mois | Gratuit 400 min/mois | Gratuit 6000 min/mois | Payant (post-2021) |
| **Config** | Jenkinsfile (Groovy) | `.github/workflows/*.yml` | `.gitlab-ci.yml` | `.circleci/config.yml` | `.travis.yml` |
| **Plugins** | 1800+ | 20 000+ actions | Intégrations natives | Orbs | Addons |
| **Communauté** | Très large (2004) | Très large (2019) | Large | Moyenne | Déclinante |
| **Enterprise** | Cloudbees (payant) | GitHub Enterprise | GitLab Enterprise | Plan team/business | N/A |
| **Pipeline as Code** | Oui | Oui | Oui | Oui | Limité |
| **GUI** | Blue Ocean | Interface GitHub | Interface GitLab | Dashboard web | Dashboard web |
| **Kubernetes natif** | Via plugin | Via actions | GitLab Runner | Via orbs | Non |
| **Idéal pour** | Grande complexité, legacy | Projets GitHub | Projets GitLab | Rapidité, performance | Migration depuis |

### Quand choisir quoi

> [!tip] Guide de décision
> - **Jenkins** : vous avez des besoins très spécifiques, une équipe DevOps dédiée, des contraintes de données on-premise, ou un legacy Jenkins existant.
> - **GitHub Actions** : projet hébergé sur GitHub, équipe qui veut démarrer rapidement, marketplace d'actions riche.
> - **GitLab CI** : stack GitLab complète, fonctionnalités DevSecOps natives (SAST, DAST, Container Scanning).
> - **CircleCI** : performances optimales, parallelism natif, ressources cloud puissantes.

---

## Jenkins — Architecture et installation

### Architecture Jenkins

```
Jenkins Master (Controller)
├── Job Queue — files d'attente des builds
├── Plugin Manager — 1800+ plugins disponibles
├── Credential Store — secrets chiffrés
├── Blue Ocean UI — interface moderne
└── Agents (Workers)
    ├── Agent permanent — machine dédiée connectée en permanence
    ├── Agent JNLP/WebSocket — connexion entrant depuis l'agent
    └── Agent éphémère — créé à la demande (Docker, Kubernetes)
```

### Installation via Docker

```bash
# Démarrer Jenkins avec Docker
docker run -d \
    --name jenkins \
    --restart unless-stopped \
    -p 8080:8080 \
    -p 50000:50000 \
    -v jenkins_home:/var/jenkins_home \
    -v /var/run/docker.sock:/var/run/docker.sock \
    jenkins/jenkins:lts

# Récupérer le mot de passe initial
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword

# Ouvrir http://localhost:8080 et entrer le mot de passe
```

### Docker Compose Jenkins + Agent

```yaml
# docker-compose.yml
version: '3.8'

services:
    jenkins:
        image: jenkins/jenkins:lts
        container_name: jenkins-master
        ports:
            - "8080:8080"
            - "50000:50000"
        volumes:
            - jenkins_home:/var/jenkins_home
            - /var/run/docker.sock:/var/run/docker.sock
        environment:
            - JAVA_OPTS=-Djenkins.install.runSetupWizard=false
        restart: unless-stopped
    
    jenkins-agent:
        image: jenkins/inbound-agent:latest
        container_name: jenkins-agent-1
        environment:
            - JENKINS_URL=http://jenkins:8080
            - JENKINS_AGENT_NAME=agent-1
            - JENKINS_SECRET=${AGENT_SECRET}
        volumes:
            - /var/run/docker.sock:/var/run/docker.sock
        depends_on:
            - jenkins
        restart: unless-stopped

volumes:
    jenkins_home:
```

### Kubernetes — Jenkins sur K8s

```yaml
# jenkins-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
    name: jenkins
    namespace: jenkins
spec:
    replicas: 1
    selector:
        matchLabels:
            app: jenkins
    template:
        metadata:
            labels:
                app: jenkins
        spec:
            serviceAccountName: jenkins
            containers:
                - name: jenkins
                  image: jenkins/jenkins:lts
                  ports:
                      - containerPort: 8080
                      - containerPort: 50000
                  volumeMounts:
                      - name: jenkins-data
                        mountPath: /var/jenkins_home
                  resources:
                      requests:
                          cpu: "500m"
                          memory: "512Mi"
                      limits:
                          cpu: "2000m"
                          memory: "2Gi"
            volumes:
                - name: jenkins-data
                  persistentVolumeClaim:
                      claimName: jenkins-pvc
```

---

## Jenkinsfile — Declarative Pipeline

### Anatomie du Declarative Pipeline

```groovy
pipeline {
    // Où s'exécute le pipeline
    agent {
        docker {
            image 'python:3.12-slim'
            args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
    }
    
    // Variables d'environnement
    environment {
        APP_NAME = 'mon-app'
        DOCKER_REGISTRY = 'registry.example.com'
        DOCKER_CREDENTIALS = credentials('docker-registry-creds')
        // credentials() injecte DOCKER_CREDENTIALS_USR et DOCKER_CREDENTIALS_PSW
    }
    
    // Options globales
    options {
        timeout(time: 30, unit: 'MINUTES')
        retry(2)
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timestamps()
        disableConcurrentBuilds()
    }
    
    // Paramètres (build paramétré)
    parameters {
        string(name: 'BRANCH', defaultValue: 'main', description: 'Branche à builder')
        booleanParam(name: 'SKIP_TESTS', defaultValue: false, description: 'Passer les tests')
        choice(name: 'ENV', choices: ['staging', 'production'], description: 'Environnement cible')
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                sh 'git log --oneline -5'
            }
        }
        
        stage('Install dependencies') {
            steps {
                sh '''
                    python -m pip install --upgrade pip
                    pip install -r requirements.txt
                    pip install -r requirements-dev.txt
                '''
            }
        }
        
        stage('Lint') {
            steps {
                sh 'flake8 src/ --max-line-length=120'
                sh 'black src/ --check'
                sh 'mypy src/'
            }
        }
        
        stage('Tests') {
            when {
                not { params.SKIP_TESTS }
            }
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh '''
                            pytest tests/unit/ -v \
                                --junit-xml=results/unit-tests.xml \
                                --cov=src --cov-report=xml:coverage.xml
                        '''
                    }
                    post {
                        always {
                            junit 'results/unit-tests.xml'
                            publishCoverage adapters: [coberturaAdapter('coverage.xml')]
                        }
                    }
                }
                stage('Integration Tests') {
                    steps {
                        sh '''
                            pytest tests/integration/ -v \
                                --junit-xml=results/integration-tests.xml
                        '''
                    }
                    post {
                        always {
                            junit 'results/integration-tests.xml'
                        }
                    }
                }
            }
        }
        
        stage('Security Scan') {
            steps {
                sh 'pip install safety bandit'
                sh 'safety check -r requirements.txt'
                sh 'bandit -r src/ -f json -o bandit-report.json'
            }
            post {
                always {
                    archiveArtifacts artifacts: 'bandit-report.json', allowEmptyArchive: true
                }
            }
        }
        
        stage('Build Docker Image') {
            when {
                branch 'main'
            }
            steps {
                script {
                    def imageTag = "${DOCKER_REGISTRY}/${APP_NAME}:${BUILD_NUMBER}"
                    def latestTag = "${DOCKER_REGISTRY}/${APP_NAME}:latest"
                    
                    docker.build(imageTag)
                    docker.withRegistry("https://${DOCKER_REGISTRY}", 'docker-registry-creds') {
                        docker.image(imageTag).push()
                        docker.image(imageTag).push('latest')
                    }
                    
                    env.IMAGE_TAG = imageTag
                }
            }
        }
        
        stage('Deploy to Staging') {
            when {
                branch 'main'
            }
            steps {
                sh """
                    kubectl set image deployment/${APP_NAME} \
                        ${APP_NAME}=${IMAGE_TAG} \
                        --namespace=staging
                    kubectl rollout status deployment/${APP_NAME} --namespace=staging --timeout=300s
                """
            }
        }
        
        stage('Approval — Production') {
            when {
                branch 'main'
                environment name: 'ENV', value: 'production'
            }
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    input message: 'Déployer en production ?',
                          ok: 'Déployer',
                          submitter: 'admin,lead-devops'
                }
            }
        }
        
        stage('Deploy to Production') {
            when {
                branch 'main'
                environment name: 'ENV', value: 'production'
            }
            steps {
                sh """
                    kubectl set image deployment/${APP_NAME} \
                        ${APP_NAME}=${IMAGE_TAG} \
                        --namespace=production
                    kubectl rollout status deployment/${APP_NAME} --namespace=production --timeout=300s
                """
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        success {
            slackSend(
                channel: '#deployments',
                color: 'good',
                message: "✅ Build ${BUILD_NUMBER} réussi — ${APP_NAME} ${GIT_BRANCH}"
            )
        }
        failure {
            slackSend(
                channel: '#deployments',
                color: 'danger',
                message: "❌ Build ${BUILD_NUMBER} échoué — ${APP_NAME} ${GIT_BRANCH}"
            )
            emailext(
                subject: "ÉCHEC: ${JOB_NAME} #${BUILD_NUMBER}",
                body: "Voir les logs: ${BUILD_URL}console",
                to: 'team@example.com'
            )
        }
        unstable {
            slackSend(channel: '#deployments', color: 'warning',
                message: "⚠️ Build instable — ${APP_NAME} ${GIT_BRANCH}")
        }
    }
}
```

### Directives `when` — Conditions

```groovy
stage('Deploy') {
    // Combiner plusieurs conditions
    when {
        allOf {
            branch 'main'                               // Sur la branche main
            not { changeRequest() }                     // Pas une PR
            environment name: 'DEPLOY', value: 'true'  // Variable env
        }
    }
    
    // Autres conditions disponibles
    when { tag "release-*" }             // Sur un tag
    when { changeset "**/backend/**" }   // Si des fichiers backend changent
    when { expression { return params.DEPLOY_PROD } }  // Expression Groovy
    when { triggeredBy 'UpstreamCause' }  // Déclenché par un job parent
}
```

### Stages parallèles

```groovy
stage('Tests parallèles') {
    parallel {
        stage('Unit') {
            agent { label 'python' }
            steps {
                sh 'pytest tests/unit/'
            }
        }
        stage('Integration') {
            agent { label 'python-db' }
            steps {
                sh 'pytest tests/integration/'
            }
        }
        stage('E2E') {
            agent { label 'browser' }
            steps {
                sh 'playwright test'
            }
        }
        stage('Security') {
            agent { label 'security' }
            steps {
                sh 'trivy fs . --exit-code 1 --severity CRITICAL'
            }
        }
    }
}
```

---

## Jenkins Shared Libraries

Les Shared Libraries permettent de **partager du code Groovy** entre plusieurs Jenkinsfiles.

### Structure

```
jenkins-shared-libs/          ← Dépôt Git séparé
├── vars/
│   ├── buildPython.groovy   ← Variable globale (appelée comme fonction)
│   ├── deployToK8s.groovy
│   └── notify.groovy
├── src/
│   └── com/example/
│       ├── Docker.groovy    ← Classe Groovy
│       └── Kubernetes.groovy
└── resources/
    └── scripts/
        └── init.sh
```

```groovy
// vars/buildPython.groovy — Variable globale
def call(Map config = [:]) {
    def pythonVersion = config.pythonVersion ?: '3.12'
    def requirements = config.requirements ?: 'requirements.txt'
    
    pipeline {
        agent {
            docker { image "python:${pythonVersion}-slim" }
        }
        stages {
            stage('Install') {
                steps {
                    sh "pip install -r ${requirements}"
                }
            }
            stage('Test') {
                steps {
                    sh 'pytest tests/ -v --junit-xml=results.xml'
                }
                post {
                    always { junit 'results.xml' }
                }
            }
        }
    }
}
```

```groovy
// vars/notify.groovy
def slack(String channel, String color, String message) {
    slackSend(channel: channel, color: color, message: message)
}

def failure(String jobName, String buildNumber) {
    slack('#alerts', 'danger', "❌ Échec: ${jobName} #${buildNumber}")
}

def success(String jobName, String buildNumber) {
    slack('#deployments', 'good', "✅ Succès: ${jobName} #${buildNumber}")
}
```

```groovy
// Jenkinsfile utilisant la shared library
@Library('jenkins-shared-libs@main') _

buildPython(
    pythonVersion: '3.11',
    requirements: 'requirements-dev.txt'
)

// Ou dans un pipeline existant
pipeline {
    stages {
        stage('Notify') {
            post {
                failure { notify.failure(JOB_NAME, BUILD_NUMBER) }
                success { notify.success(JOB_NAME, BUILD_NUMBER) }
            }
        }
    }
}
```

---

## Plugins Jenkins essentiels

| Plugin | Utilité |
|--------|---------|
| **Git** | Intégration Git (inclus par défaut) |
| **Pipeline** | Support Jenkinsfile declarative/scripted |
| **Blue Ocean** | UI moderne pour les pipelines |
| **Docker Pipeline** | `docker.build()`, `docker.withRegistry()` |
| **Kubernetes** | Agents éphémères sur K8s |
| **Credentials Binding** | Injection sécurisée de secrets |
| **Slack Notification** | Notifications Slack |
| **JUnit** | Publication des résultats de tests |
| **SonarQube Scanner** | Analyse qualité de code |
| **OWASP Dependency-Check** | Scan CVE des dépendances |
| **Blue Ocean** | Interface graphique moderne |
| **GitHub Branch Source** | Multibranch pipelines depuis GitHub |
| **Multibranch Scan Webhook** | Trigger sur PR GitHub/GitLab |

---

## GitLab CI

### Anatomie de `.gitlab-ci.yml`

```yaml
# .gitlab-ci.yml
image: python:3.12-slim

# Variables globales
variables:
    PIP_CACHE_DIR: "$CI_PROJECT_DIR/.cache/pip"
    DOCKER_DRIVER: overlay2
    APP_NAME: mon-app

# Cache entre jobs
cache:
    key: ${CI_COMMIT_REF_SLUG}
    paths:
        - .cache/pip
        - venv/

# Définition des stages (ordre d'exécution)
stages:
    - prepare
    - test
    - build
    - scan
    - deploy
    - notify

# ── PREPARE ──
install:
    stage: prepare
    script:
        - pip install virtualenv
        - virtualenv venv
        - source venv/bin/activate
        - pip install -r requirements.txt -r requirements-dev.txt
    artifacts:
        paths:
            - venv/
        expire_in: 1 hour

# ── TEST ──
lint:
    stage: test
    script:
        - source venv/bin/activate
        - flake8 src/
        - black src/ --check
    needs: [install]

unit-tests:
    stage: test
    script:
        - source venv/bin/activate
        - pytest tests/unit/ -v --junit-xml=unit-results.xml --cov=src
    artifacts:
        when: always
        reports:
            junit: unit-results.xml
        paths:
            - coverage.xml
    needs: [install]
    coverage: '/TOTAL.*\s+(\d+%)$/'

integration-tests:
    stage: test
    services:
        - postgres:15
        - redis:7
    variables:
        DATABASE_URL: postgresql://postgres:postgres@postgres/test
        REDIS_URL: redis://redis:6379
    script:
        - source venv/bin/activate
        - pytest tests/integration/ -v --junit-xml=integration-results.xml
    artifacts:
        when: always
        reports:
            junit: integration-results.xml
    needs: [install]

# ── BUILD ──
build-docker:
    stage: build
    image: docker:24
    services:
        - docker:24-dind
    script:
        - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
        - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
        - docker tag $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA $CI_REGISTRY_IMAGE:latest
        - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
        - docker push $CI_REGISTRY_IMAGE:latest
    rules:
        - if: $CI_COMMIT_BRANCH == "main"

# ── SCAN ──
security-scan:
    stage: scan
    image: 
        name: aquasec/trivy:latest
        entrypoint: [""]
    script:
        - trivy image --exit-code 0 --severity HIGH,CRITICAL $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
    needs: [build-docker]
    allow_failure: true

dependency-check:
    stage: scan
    script:
        - pip install safety
        - safety check -r requirements.txt --json > safety-report.json
    artifacts:
        when: always
        paths:
            - safety-report.json

# ── DEPLOY ──
deploy-staging:
    stage: deploy
    image: bitnami/kubectl:latest
    environment:
        name: staging
        url: https://staging.example.com
    script:
        - echo "$KUBE_CONFIG_STAGING" | base64 -d > kubeconfig
        - export KUBECONFIG=kubeconfig
        - kubectl set image deployment/$APP_NAME $APP_NAME=$CI_REGISTRY_IMAGE:$CI_COMMIT_SHA -n staging
        - kubectl rollout status deployment/$APP_NAME -n staging --timeout=300s
    rules:
        - if: $CI_COMMIT_BRANCH == "main"
    needs: [build-docker, security-scan]

deploy-production:
    stage: deploy
    image: bitnami/kubectl:latest
    environment:
        name: production
        url: https://app.example.com
    script:
        - echo "$KUBE_CONFIG_PROD" | base64 -d > kubeconfig
        - export KUBECONFIG=kubeconfig
        - kubectl set image deployment/$APP_NAME $APP_NAME=$CI_REGISTRY_IMAGE:$CI_COMMIT_SHA -n production
        - kubectl rollout status deployment/$APP_NAME -n production --timeout=300s
    when: manual          # Approbation manuelle
    allow_failure: false
    rules:
        - if: $CI_COMMIT_BRANCH == "main"
    needs: [deploy-staging]
```

### DAG avec `needs`

```yaml
# Exécution en parallèle et avec dépendances précises (pas juste par stage)
build-frontend:
    stage: build
    script: npm run build
    needs: []   # Peut démarrer dès que possible

build-backend:
    stage: build
    script: docker build -t backend .
    needs: [unit-tests]

deploy-staging:
    stage: deploy
    script: ./deploy.sh staging
    needs: [build-frontend, build-backend]  # Attend les deux
```

### Rules — contrôle fin de l'exécution

```yaml
deploy-prod:
    rules:
        # Sur main, seulement si pas un merge request
        - if: $CI_COMMIT_BRANCH == "main" && $CI_PIPELINE_SOURCE != "merge_request_event"
          when: manual
        # Sur les tags de release
        - if: $CI_COMMIT_TAG =~ /^v\d+\.\d+\.\d+$/
          when: on_success
        # Sinon, ne pas exécuter
        - when: never
```

---

## CircleCI

### Anatomie de `.circleci/config.yml`

```yaml
# .circleci/config.yml
version: 2.1

# Orbs — packages réutilisables (équivalent aux actions GitHub)
orbs:
    python: circleci/python@2.1.1
    docker: circleci/docker@2.4.0
    kubernetes: circleci/kubernetes@1.3.1
    slack: circleci/slack@4.12.5
    sonarcloud: sonarsource/sonarcloud@2.0.0

# Paramètres du pipeline
parameters:
    run-integration-tests:
        type: boolean
        default: false

# Exécuteurs réutilisables
executors:
    python-executor:
        docker:
            - image: cimg/python:3.12
        resource_class: medium
    
    python-with-db:
        docker:
            - image: cimg/python:3.12
            - image: cimg/postgres:15.0
              environment:
                  POSTGRES_USER: test
                  POSTGRES_DB: testdb
                  POSTGRES_PASSWORD: testpass
        resource_class: medium+

# Commandes réutilisables
commands:
    setup-python:
        steps:
            - python/install-packages:
                pkg-manager: pip
                packages: requirements.txt requirements-dev.txt
    
    run-tests:
        parameters:
            test-dir:
                type: string
                default: tests/
        steps:
            - run:
                name: Exécuter les tests
                command: |
                    pytest << parameters.test-dir >> -v \
                        --junit-xml=test-results/results.xml \
                        --cov=src --cov-report=xml

jobs:
    lint:
        executor: python-executor
        steps:
            - checkout
            - setup-python
            - run: flake8 src/ --max-line-length=120
            - run: black src/ --check
            - run: mypy src/

    unit-tests:
        executor: python-executor
        parallelism: 4  # Distribuer les tests sur 4 conteneurs
        steps:
            - checkout
            - setup-python
            - run:
                name: Découper les tests (parallelism)
                command: |
                    TESTFILES=$(circleci tests glob "tests/unit/**/*.py" | \
                        circleci tests split --split-by=timings)
                    pytest $TESTFILES -v --junit-xml=test-results/results.xml
            - store_test_results:
                path: test-results
            - store_artifacts:
                path: coverage.xml

    integration-tests:
        executor: python-with-db
        steps:
            - checkout
            - setup-python
            - run:
                environment:
                    DATABASE_URL: postgresql://test:testpass@localhost/testdb
                command: pytest tests/integration/ -v --junit-xml=results.xml
            - store_test_results:
                path: .

    build-and-push:
        executor: docker/docker
        steps:
            - checkout
            - setup_remote_docker:
                docker_layer_caching: true
            - docker/check
            - docker/build:
                image: $DOCKER_LOGIN/$CIRCLE_PROJECT_REPONAME
                tag: $CIRCLE_SHA1
            - docker/push:
                image: $DOCKER_LOGIN/$CIRCLE_PROJECT_REPONAME
                tag: $CIRCLE_SHA1

    deploy-staging:
        executor: python-executor
        steps:
            - kubernetes/install-kubectl
            - run:
                name: Deploy to staging
                command: |
                    echo "$KUBE_CONFIG" | base64 -d > kubeconfig
                    export KUBECONFIG=kubeconfig
                    kubectl set image deployment/mon-app \
                        mon-app=$DOCKER_LOGIN/$CIRCLE_PROJECT_REPONAME:$CIRCLE_SHA1 \
                        -n staging

# Workflows — orchestration des jobs
workflows:
    ci-cd:
        jobs:
            - lint
            
            - unit-tests:
                requires: [lint]
            
            - integration-tests:
                requires: [lint]
                # Seulement sur main et si paramètre activé
                filters:
                    branches:
                        only: main
            
            - build-and-push:
                requires: [unit-tests, integration-tests]
                filters:
                    branches:
                        only: main
                context: docker-hub  # Secrets partagés
            
            - deploy-staging:
                requires: [build-and-push]
                filters:
                    branches:
                        only: main
            
            # Approbation manuelle avant production
            - approve-production:
                type: approval
                requires: [deploy-staging]
            
            - deploy-production:
                requires: [approve-production]
                context: production-k8s
```

---

## Patterns CI/CD

### Pipeline complète — Test → Build → Scan → Sign → Deploy

```yaml
# .github/workflows/complete-pipeline.yml
name: Complete CI/CD Pipeline

on:
    push:
        branches: [main]
    pull_request:
        branches: [main]

jobs:
    test:
        runs-on: ubuntu-latest
        steps:
            - uses: actions/checkout@v4
            - uses: actions/setup-python@v5
              with:
                  python-version: '3.12'
                  cache: pip
            - run: pip install -r requirements-dev.txt
            - run: pytest tests/ --junit-xml=results.xml --cov=src
            - uses: actions/upload-artifact@v4
              with:
                  name: test-results
                  path: results.xml

    lint:
        runs-on: ubuntu-latest
        steps:
            - uses: actions/checkout@v4
            - uses: actions/setup-python@v5
              with:
                  python-version: '3.12'
                  cache: pip
            - run: pip install flake8 black mypy
            - run: flake8 src/ && black src/ --check && mypy src/

    build:
        needs: [test, lint]
        runs-on: ubuntu-latest
        if: github.event_name == 'push' && github.ref == 'refs/heads/main'
        outputs:
            image-digest: ${{ steps.build.outputs.digest }}
        steps:
            - uses: actions/checkout@v4
            
            - name: Log in to Container Registry
              uses: docker/login-action@v3
              with:
                  registry: ghcr.io
                  username: ${{ github.actor }}
                  password: ${{ secrets.GITHUB_TOKEN }}
            
            - name: Build and push
              id: build
              uses: docker/build-push-action@v5
              with:
                  context: .
                  push: true
                  tags: |
                      ghcr.io/${{ github.repository }}:${{ github.sha }}
                      ghcr.io/${{ github.repository }}:latest
                  cache-from: type=gha
                  cache-to: type=gha,mode=max

    scan:
        needs: build
        runs-on: ubuntu-latest
        steps:
            - name: Scan image with Trivy
              uses: aquasecurity/trivy-action@master
              with:
                  image-ref: ghcr.io/${{ github.repository }}:${{ github.sha }}
                  format: sarif
                  output: trivy-results.sarif
                  severity: CRITICAL,HIGH
                  exit-code: 1
            
            - uses: github/codeql-action/upload-sarif@v3
              with:
                  sarif_file: trivy-results.sarif

    sign:
        needs: scan
        runs-on: ubuntu-latest
        steps:
            - name: Install Cosign
              uses: sigstore/cosign-installer@v3
            
            - name: Sign container image
              run: |
                  cosign sign --yes ghcr.io/${{ github.repository }}@${{ needs.build.outputs.image-digest }}
              env:
                  COSIGN_EXPERIMENTAL: 1  # Keyless signing

    deploy-staging:
        needs: sign
        environment: staging
        runs-on: ubuntu-latest
        steps:
            - uses: actions/checkout@v4
            - uses: azure/setup-kubectl@v3
            - run: |
                  echo "${{ secrets.KUBE_CONFIG_STAGING }}" | base64 -d > kubeconfig
                  KUBECONFIG=kubeconfig kubectl set image deployment/mon-app \
                      mon-app=ghcr.io/${{ github.repository }}:${{ github.sha }} \
                      -n staging
                  KUBECONFIG=kubeconfig kubectl rollout status deployment/mon-app -n staging

    deploy-production:
        needs: deploy-staging
        environment: production  # Environnement avec approbation requise dans GitHub
        runs-on: ubuntu-latest
        steps:
            - uses: actions/checkout@v4
            - uses: azure/setup-kubectl@v3
            - run: |
                  echo "${{ secrets.KUBE_CONFIG_PROD }}" | base64 -d > kubeconfig
                  KUBECONFIG=kubeconfig kubectl set image deployment/mon-app \
                      mon-app=ghcr.io/${{ github.repository }}:${{ github.sha }} \
                      -n production
```

### Blue/Green Deployment

```bash
#!/bin/bash
# blue-green-deploy.sh

NAMESPACE="production"
APP="mon-app"
NEW_IMAGE="ghcr.io/myorg/mon-app:${IMAGE_TAG}"

# Déterminer la couleur active
CURRENT=$(kubectl get service $APP -n $NAMESPACE -o jsonpath='{.spec.selector.color}')
if [ "$CURRENT" == "blue" ]; then
    NEW_COLOR="green"
else
    NEW_COLOR="blue"
fi

echo "Déploiement: $CURRENT → $NEW_COLOR"

# Déployer la nouvelle version (couleur inactive)
kubectl set image deployment/$APP-$NEW_COLOR \
    $APP=$NEW_IMAGE \
    -n $NAMESPACE

# Attendre la readiness
kubectl rollout status deployment/$APP-$NEW_COLOR -n $NAMESPACE --timeout=300s

# Sanity check (smoke test)
ENDPOINT=$(kubectl get service $APP-$NEW_COLOR -n $NAMESPACE -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
if curl -f "http://$ENDPOINT/health" ; then
    echo "Smoke test OK — Basculement du trafic vers $NEW_COLOR"
    
    # Basculer le service principal vers la nouvelle couleur
    kubectl patch service $APP -n $NAMESPACE \
        -p "{\"spec\":{\"selector\":{\"color\":\"$NEW_COLOR\"}}}"
    
    echo "✅ Déploiement $NEW_COLOR réussi"
else
    echo "❌ Smoke test échoué — Rollback"
    exit 1
fi
```

### Canary Deployment

```yaml
# canary-deploy.yaml (avec NGINX Ingress)
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
    name: mon-app-canary
    annotations:
        nginx.ingress.kubernetes.io/canary: "true"
        nginx.ingress.kubernetes.io/canary-weight: "10"  # 10% du trafic
spec:
    rules:
        - host: mon-app.example.com
          http:
              paths:
                  - path: /
                    pathType: Prefix
                    backend:
                        service:
                            name: mon-app-canary
                            port:
                                number: 80
```

### Feature Flags en pipeline

```python
# feature_flags.py
import os

class FeatureFlags:
    # Lire depuis env (injecté par CI/CD)
    NEW_CHECKOUT_FLOW = os.getenv('FF_NEW_CHECKOUT', 'false').lower() == 'true'
    DARK_MODE = os.getenv('FF_DARK_MODE', 'true').lower() == 'true'
    BETA_DASHBOARD = os.getenv('FF_BETA_DASHBOARD', 'false').lower() == 'true'

# Dans le pipeline (GitHub Actions)
# env:
#   FF_NEW_CHECKOUT: ${{ vars.FF_NEW_CHECKOUT }}  # Variables d'environnement GitHub
```

---

## Self-hosted Runners

### GitHub Actions Self-Hosted Runner

```bash
# Sur la machine runner
mkdir actions-runner && cd actions-runner

# Télécharger le runner (adapter la version et l'OS)
curl -o actions-runner-linux-x64-2.314.1.tar.gz -L \
    https://github.com/actions/runner/releases/download/v2.314.1/actions-runner-linux-x64-2.314.1.tar.gz

tar xzf ./actions-runner-linux-x64-2.314.1.tar.gz

# Configurer (token depuis GitHub Settings → Actions → Runners → New runner)
./config.sh --url https://github.com/myorg/myrepo --token TOKEN123

# Installer comme service
sudo ./svc.sh install
sudo ./svc.sh start
```

```yaml
# Utiliser dans un workflow
jobs:
    build:
        runs-on: self-hosted  # Ou un label custom
        # runs-on: [self-hosted, linux, gpu]  # Labels multiples
```

### GitLab Runner sur Kubernetes

```yaml
# gitlab-runner-values.yaml (Helm chart)
gitlabUrl: https://gitlab.example.com
runnerRegistrationToken: "YOUR_TOKEN"

runners:
    config: |
        [[runners]]
            [runners.kubernetes]
                namespace = "gitlab-runner"
                image = "ubuntu:22.04"
                cpu_limit = "2"
                memory_limit = "2Gi"
                cpu_request = "500m"
                memory_request = "512Mi"
            [runners.kubernetes.volumes.empty_dir]
                name = "docker-certs"
                mount_path = "/certs/client"
                medium = "Memory"
```

```bash
helm repo add gitlab https://charts.gitlab.io
helm install gitlab-runner gitlab/gitlab-runner \
    -f gitlab-runner-values.yaml \
    -n gitlab-runner \
    --create-namespace
```

---

## Sécurité CI/CD

### Gestion des secrets

#### Jenkins Credentials Store
```groovy
// Jenkinsfile — Utilisation sécurisée des credentials
pipeline {
    environment {
        // Credentials binding
        AWS_CREDS = credentials('aws-production')
        // Injecte AWS_CREDS_ACCESS_KEY_ID et AWS_CREDS_SECRET_ACCESS_KEY
        
        DB_PASSWORD = credentials('db-password-prod')
        // Injecte DB_PASSWORD comme variable masquée dans les logs
    }
    stages {
        stage('Deploy') {
            steps {
                withCredentials([
                    string(credentialsId: 'stripe-api-key', variable: 'STRIPE_KEY'),
                    file(credentialsId: 'google-service-account', variable: 'GOOGLE_SA_FILE')
                ]) {
                    sh 'deploy.sh'  // STRIPE_KEY disponible, masqué dans les logs
                }
            }
        }
    }
}
```

#### HashiCorp Vault dans les pipelines

```yaml
# GitHub Actions avec Vault
- name: Import Secrets from Vault
  uses: hashicorp/vault-action@v2
  with:
      url: https://vault.example.com
      token: ${{ secrets.VAULT_TOKEN }}
      secrets: |
          secret/data/production db_password | DB_PASSWORD ;
          secret/data/production api_key | API_KEY

- name: Deploy
  env:
      DB_PASSWORD: ${{ env.DB_PASSWORD }}
  run: ./deploy.sh
```

### Scan des secrets dans le code

```yaml
# GitHub Actions — Gitleaks (scan de secrets dans le code)
- name: Gitleaks Secret Scan
  uses: gitleaks/gitleaks-action@v2
  env:
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      GITLEAKS_LICENSE: ${{ secrets.GITLEAKS_LICENSE }}

# Configuration .gitleaks.toml
[allowlist]
    description = "Exceptions autorisées"
    regexes = ['''test-token-.*''']  # Tokens de test
    paths = ['''(^|/)tests?/''']     # Fichiers de test
```

### SAST en pipeline

```yaml
# GitHub Actions — CodeQL (SAST)
- name: Initialize CodeQL
  uses: github/codeql-action/init@v3
  with:
      languages: python, javascript

- name: Autobuild
  uses: github/codeql-action/autobuild@v3

- name: Analyze
  uses: github/codeql-action/analyze@v3
  with:
      category: "/language:python"

# GitLab CI — SAST natif
include:
    - template: Security/SAST.gitlab-ci.yml
    - template: Security/Dependency-Scanning.gitlab-ci.yml
    - template: Security/Container-Scanning.gitlab-ci.yml
```

### Politique de sécurité minimale (OSSF Scorecard)

```yaml
# .github/workflows/scorecard.yml — Score OpenSSF
name: Scorecard supply-chain security
on:
    schedule:
        - cron: '0 8 * * 1'  # Chaque lundi à 8h
    push:
        branches: [main]

jobs:
    analysis:
        runs-on: ubuntu-latest
        steps:
            - uses: actions/checkout@v4
            - uses: ossf/scorecard-action@v2.3.1
              with:
                  results_format: sarif
                  results_file: results.sarif
            - uses: github/codeql-action/upload-sarif@v3
              with:
                  sarif_file: results.sarif
```

---

## Exercices Pratiques

### Exercice 1 — Jenkinsfile complet

Créez un `Jenkinsfile` pour une API Node.js/Express :
- **Stage Lint** : ESLint + Prettier
- **Stage Test** : Jest avec rapport JUnit et couverture
- **Stage Build** : Image Docker taguée avec `${GIT_COMMIT_SHORT}`
- **Stage Scan** : Trivy sur l'image Docker (bloquer si CVE CRITICAL)
- **Stage Deploy** : SSH vers un serveur staging (port 22, utiliser les credentials Jenkins)
- **Post** : Notification Slack succès/échec avec lien vers le build

### Exercice 2 — GitLab CI pour microservices

Vous avez un monorepo avec 3 services : `frontend/`, `api/`, `worker/`.
Créez un `.gitlab-ci.yml` qui :
- Ne build/teste/déploie que les services **modifiés** dans le commit (utilisez `changes:` dans les rules)
- Lance les tests des 3 services **en parallèle** si tous sont modifiés
- Utilise `needs:` pour créer un DAG précis (pas juste des stages)
- Déploie en staging automatiquement, production avec approbation manuelle

### Exercice 3 — CircleCI avec orbs

Créez un `.circleci/config.yml` pour une app React + API FastAPI :
- Orb Python pour le backend, Node pour le frontend
- Tests frontend (Vitest) et backend (pytest) en parallèle
- Build Docker multi-stage (frontend compilé, intégré dans le conteneur backend)
- Parallelism CircleCI pour les tests backend (4 conteneurs)
- Déploiement sur Heroku via l'orb officiel

### Exercice 4 — Blue/Green sur Kubernetes

Implémentez un blue/green deployment :
1. Deux Deployments K8s : `app-blue` et `app-green`, initialement blue actif
2. Script bash qui : détecte la couleur active, déploie sur l'inactive, fait un smoke test HTTP, bascule le Service, rollback si le test échoue
3. GitHub Actions workflow qui appelle ce script
4. Rollback automatique si le rollout status échoue

### Exercice 5 — Sécurité CI complète

Ajoutez une pipeline de sécurité complète à un projet Python existant :
- **Gitleaks** : scan des secrets committeés (bloquer le commit)
- **Bandit** : SAST Python (rapport HTML en artefact)
- **Safety** : scan CVE des dépendances Python (bloquer si HIGH+)
- **Trivy** : scan image Docker (SARIF uploadé dans GitHub Security)
- **OSSF Scorecard** : exécuté chaque semaine sur un cron
- Toutes ces étapes **en parallèle** pour ne pas allonger le pipeline principal

---

## Liens et Références

- [[04 - CI-CD avec GitHub Actions]] — GitHub Actions en détail
- [[07 - DevSecOps]] — Sécurité intégrée dans la CI/CD
- [[01 - Docker]] — Conteneurs utilisés dans les pipelines
- [[04 - Kubernetes Introduction]] — Déploiement sur Kubernetes depuis la CI
