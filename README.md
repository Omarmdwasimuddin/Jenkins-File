# Jenkinsfile Complete Guide
> Jenkins CI/CD Pipeline Explain

---

## ১. Jenkinsfile কি?

Jenkinsfile হলো একটি text file যা Jenkins-এর CI/CD pipeline কে **code আকারে** define করে। এটি সাধারণত project-এর root directory-তে রাখা হয় এবং Git repository-তে commit করা হয়।

এটি দুটি format-এ লেখা যায়:

- **Declarative Pipeline** — সহজ, structured format (নতুনদের জন্য recommended)
- **Scripted Pipeline** — Groovy-based, বেশি flexible কিন্তু complex

---

## ২. মূল Structure

একটি Declarative Jenkinsfile-এর basic structure:

```groovy
pipeline {
    agent any

    environment {
        APP_NAME = 'my-app'
        VERSION  = '1.0.0'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Deploy') {
            when { branch 'main' }
            steps {
                sh './deploy.sh'
            }
        }
    }

    post {
        success { slackSend message: 'Build সফল!' }
        failure { mail to: 'team@example.com', subject: 'Build ব্যর্থ!' }
    }
}
```

---

## ৩. গুরুত্বপূর্ণ Block-গুলোর ব্যাখ্যা

### `agent`

Pipeline কোন machine বা container-এ run করবে তা নির্ধারণ করে।

| Agent Option | ব্যাখ্যা |
|---|---|
| `agent any` | যেকোনো available Jenkins node-এ run করবে |
| `agent none` | প্রতিটি stage নিজের agent define করবে |
| `agent { docker 'node:18' }` | Docker container-এ run করবে |
| `agent { label 'linux' }` | নির্দিষ্ট label-এর node-এ run করবে |

---

### `environment`

Global environment variables এবং secrets define করার জায়গা।

```groovy
environment {
    APP_NAME    = 'my-application'
    BUILD_ENV   = 'production'
    DB_PASSWORD = credentials('db-secret-id')  // Jenkins credential
}
```

---

### `stages` ও `stage`

Pipeline-এর প্রতিটি ধাপ `stage` দিয়ে define করা হয়। সব stage মিলে `stages` block তৈরি হয়।

```groovy
stages {
    stage('Stage নাম') {
        steps {
            // এখানে commands লেখা হয়
            sh 'echo Hello'
        }
    }
}
```

---

### `when` — Conditional Execution

নির্দিষ্ট condition পূরণ হলেই শুধু stage run হবে।

```groovy
stage('Deploy') {
    when {
        branch 'main'          // শুধু main branch-এ
        // environment name: 'APP', value: 'prod'
        // expression { return params.DEPLOY == 'yes' }
    }
    steps { sh './deploy.sh' }
}
```

---

### `post` — Pipeline-এর পরে

Pipeline শেষ হওয়ার পরে কোন action নেবে তা define করে।

| Post Condition | কখন চলে |
|---|---|
| `always` | সবসময়, success বা failure যাই হোক |
| `success` | শুধু build সফল হলে |
| `failure` | শুধু build ব্যর্থ হলে |
| `unstable` | Build unstable হলে (test fail) |
| `changed` | আগের build-এর status পরিবর্তন হলে |

```groovy
post {
    always   { cleanWs() }
    success  { slackSend color: 'good', message: 'Build সফল!' }
    failure  { mail to: 'devops@company.com', subject: 'Build ব্যর্থ!' }
}
```

---

## ৪. অন্যান্য গুরুত্বপূর্ণ Block

### `parameters` — User Input

Build চালানোর সময় user-এর কাছ থেকে input নেওয়া যায়।

```groovy
parameters {
    string(name: 'VERSION', defaultValue: '1.0', description: 'Version number')
    booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Run tests?')
    choice(name: 'ENV', choices: ['dev', 'staging', 'prod'], description: 'Environment')
}
```

---

### `triggers` — Automatic Build

কখন automatically build হবে তা define করে।

```groovy
triggers {
    cron('H 4 * * 1-5')       // প্রতি weekday রাত ৪টায়
    pollSCM('H/5 * * * *')    // প্রতি ৫ মিনিটে SCM check
    upstream(upstreamProjects: 'other-job', threshold: hudson.model.Result.SUCCESS)
}
```

---

### `parallel` — একসাথে একাধিক Stage

একাধিক stage একই সাথে parallel-এ চালানো যায়, যা build time কমায়।

```groovy
stage('Parallel Tests') {
    parallel {
        stage('Unit Tests') {
            steps { sh 'mvn test -Dtest=Unit*' }
        }
        stage('Integration Tests') {
            steps { sh 'mvn test -Dtest=Integration*' }
        }
    }
}
```

---

## ৫. Declarative vs Scripted Pipeline

| বৈশিষ্ট্য | Declarative | Scripted |
|---|---|---|
| Syntax | `pipeline { }` block | `node { }` block |
| সহজতা | সহজ, structured | Complex, Groovy-based |
| Flexibility | কম flexible | বেশি flexible |
| Recommended for | নতুনদের জন্য | Advanced use case |

---

## ৬. সম্পূর্ণ উদাহরণ — Real-world Pipeline

একটি Node.js application-এর complete Jenkinsfile:

```groovy
pipeline {
    agent { docker { image 'node:18-alpine' } }

    environment {
        APP_NAME      = 'node-app'
        DOCKER_IMAGE  = "myregistry/${APP_NAME}:${BUILD_NUMBER}"
        REGISTRY_CRED = credentials('docker-registry')
    }

    parameters {
        choice(
            name: 'DEPLOY_ENV',
            choices: ['dev', 'staging', 'prod'],
            description: 'Deploy environment'
        )
    }

    triggers { cron('H 2 * * 1-5') }

    stages {
        stage('Checkout') {
            steps { checkout scm }
        }

        stage('Install') {
            steps { sh 'npm ci' }
        }

        stage('Test') {
            parallel {
                stage('Unit') { steps { sh 'npm run test:unit' } }
                stage('Lint') { steps { sh 'npm run lint' } }
            }
        }

        stage('Build Docker') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE} ."
                sh "docker push ${DOCKER_IMAGE}"
            }
        }

        stage('Deploy') {
            when { expression { params.DEPLOY_ENV != 'dev' } }
            steps {
                sh "./deploy.sh ${params.DEPLOY_ENV} ${DOCKER_IMAGE}"
            }
        }
    }

    post {
        always  { cleanWs() }
        success { slackSend color: 'good', message: "Build #${BUILD_NUMBER} সফল!" }
        failure { mail to: 'devops@company.com', subject: "Build #${BUILD_NUMBER} ব্যর্থ!" }
    }
}
```

---

*— Jenkinsfile Guide | Nazim Interprise DevOps Team —*
