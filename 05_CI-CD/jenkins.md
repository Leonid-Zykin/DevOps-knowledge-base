tags: [devops, jenkins]

## Jenkins — Jenkinsfile, агенты, shared libraries

> ⚡ Tip: Jenkins pipelines описываются в `Jenkinsfile` (declarative/ scripted). Держите их в Git и по возможности используйте declarative pipeline.

### Базовый declarative Jenkinsfile

```groovy
pipeline {
  agent any

  options {
    timestamps()
    disableConcurrentBuilds()
  }

  environment {
    APP_ENV = 'dev'
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Test') {
      agent { docker { image 'python:3.11' } }
      steps {
        sh 'pip install -r requirements.txt'
        sh 'pytest -q'
      }
    }

    stage('Build image') {
      steps {
        sh '''
          docker build -t registry.example.com/app:${BUILD_NUMBER} .
          docker push registry.example.com/app:${BUILD_NUMBER}
        '''
      }
    }
  }
}
```

### Агенты

- `agent any` — любой доступный executor;
- `agent { label 'docker' }` — ноды с лейблом `docker`;
- `agent { docker { image 'node:20' } }` — Docker‑agent.

Примеры:

```groovy
pipeline {
  agent none
  stages {
    stage('Lint') {
      agent { docker { image 'node:20' } }
      steps {
        sh 'npm ci'
        sh 'npm run lint'
      }
    }
  }
}
```

### Настройка stages, steps, post

```groovy
pipeline {
  agent any
  stages {
    stage('Build') {
      steps {
        sh './gradlew build'
        archiveArtifacts artifacts: 'build/libs/*.jar', fingerprint: true
      }
    }
  }
  post {
    success {
      emailext subject: "Build #${env.BUILD_NUMBER} SUCCESS",
               to: 'team@example.com',
               body: "Job ${env.JOB_NAME} succeeded."
    }
    failure {
      emailext subject: "Build #${env.BUILD_NUMBER} FAILED",
               to: 'team@example.com',
               body: "Job ${env.JOB_NAME} failed."
    }
  }
}
```

### Shared Libraries

1. Создать репозиторий для shared lib (структура `vars/`, `src/`).
2. В Jenkins: Configure System → Global Pipeline Libraries.

Пример `vars/deployApp.groovy`:

```groovy
def call(Map args = [:]) {
  def envName = args.get('env', 'dev')
  sh """
    ./deploy.sh ${envName}
  """
}
```

Использование в Jenkinsfile:

```groovy
@Library('my-shared-lib') _

pipeline {
  agent any
  stages {
    stage('Deploy') {
      steps {
        deployApp(env: 'prod')
      }
    }
  }
}
```

### Credentials

В Jenkins: Manage Credentials → добавить (username/password, secret text, SSH‑ключ).

Использование:

```groovy
withCredentials([usernamePassword(credentialsId: 'docker-creds',
                                  usernameVariable: 'DOCKER_USER',
                                  passwordVariable: 'DOCKER_PASS')]) {
  sh '''
    echo "$DOCKER_PASS" | docker login registry.example.com \
      --username "$DOCKER_USER" --password-stdin
  '''
}
```

### Интеграции

- Docker: см. `[[03_Docker/docker-cheatsheet]]`, `[[03_Docker/dockerfile-best-practices]]`;
- Kubernetes: Jenkins + kubectl/Helm/Argo CD (`[[04_Kubernetes/kubectl-cheatsheet]]`, `[[04_Kubernetes/helm]]`, `[[argocd]]`);
- Terraform: `[[06_Terraform/terraform-cheatsheet]]`, `[[06_Terraform/state-management]]`.

## Gotchas

- **Scripted pipeline вместо declarative без нужды**  
  - **Проблема**: сложнее читать, больше boilerplate, труднее поддерживать.  
  - **Решение**: использовать declarative pipelines, scripted — только для очень специфических случаев.

- **Логика деплоя, размазанная по нескольким Jenkinsfile**  
  - **Проблема**: сложно переиспользовать и изменять, дублирование.  
  - **Решение**: выносить общий код в shared libraries, Jenkinsfile держать тонким.

- **Секреты в Jenkinsfile**  
  - **Проблема**: секреты в Git, утечка при просмотре логов.  
  - **Решение**: использовать credentials store Jenkins и `withCredentials`, не логировать значения.

> ⚠️ Warning: Jenkins‑master и его хранилище credentials — один из самых критичных компонентов инфраструктуры. Компрометация Jenkins даёт атакующему ключи от CI/CD, реестров, кластеров и облаков. Ограничивайте доступ, делайте регулярные бэкапы и обновления, используйте RBAC и изолированные агенты.

