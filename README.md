# Jenkins Pipeline — Practical Guide
> Declarative ও Scripted Pipeline 

---

## ১. Declarative Pipeline

### ধাপসমূহ

1. Jenkins-এ **New Item** click করো
2. **Enter an item name:** `Declarative-Pipeline`
3. **Select:** `Pipeline`
4. **Click:** `OK`
5. **Pipeline** section-এ নিচের Script দাও
6. **Save** click করো
7. **Console Output** দেখো

### Script

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building the project....'
            }
        }
        stage('Test') {
            steps {
                echo 'Running test....'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying the App....'
            }
        }
    }
}
```

### Console Output-এ যা দেখবে

```
Started by user admin
[Pipeline] Start of Pipeline
[Pipeline] agent any
[Pipeline] stage (Build)
[Pipeline] echo
Building the project....
[Pipeline] stage (Test)
[Pipeline] echo
Running test....
[Pipeline] stage (Deploy)
[Pipeline] echo
Deploying the App....
[Pipeline] End of Pipeline
Finished: SUCCESS
```

---

## ২. Scripted Pipeline

### ধাপসমূহ

1. Jenkins-এ **New Item** click করো
2. **Enter an item name:** `Scripted-Pipeline`
3. **Select:** `Pipeline`
4. **Click:** `OK`
5. **Pipeline** section-এ নিচের Script দাও
6. **Save** click করো
7. **Console Output** দেখো

### Script

```groovy
node {
    stage('Build') {
        echo 'Building the project....'
    }
    stage('Test') {
        echo 'Testing the project....'
    }
    stage('Deploy') {
        echo 'Deploying the project....'
    }
}
```

### Console Output-এ যা দেখবে

```
Started by user admin
[Pipeline] Start of Pipeline
[Pipeline] node
Running on Jenkins in /var/jenkins_home/workspace/Scripted-Pipeline
[Pipeline] stage (Build)
[Pipeline] echo
Building the project....
[Pipeline] stage (Test)
[Pipeline] echo
Testing the project....
[Pipeline] stage (Deploy)
[Pipeline] echo
Deploying the project....
[Pipeline] End of Pipeline
Finished: SUCCESS
```

---

## ৩. Declarative vs Scripted — পার্থক্য

| বিষয় | Declarative | Scripted |
|---|---|---|
| শুরু হয় | `pipeline { }` দিয়ে | `node { }` দিয়ে |
| `steps { }` লাগে? | হ্যাঁ, প্রতিটি stage-এ | না, সরাসরি commands |
| সহজতা | বেশি সহজ ও structured | একটু complex |
| Syntax error ধরে | Build-এর আগেই | Runtime-এ |
| Recommended | নতুনদের জন্য ✅ | Advanced user-দের জন্য |

---

## ৪. মূল পার্থক্য Code-এ

```groovy
// ✅ Declarative — steps { } ব্লক আবশ্যক
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {           // <-- এটা লাগবেই
                echo 'Building....'
            }
        }
    }
}

// ✅ Scripted — steps { } ছাড়াই চলে
node {
    stage('Build') {
        echo 'Building....'   // <-- সরাসরি command
    }
}
```

---

## ৫. Pipeline Stage View

Build সফল হলে Jenkins Dashboard-এ এরকম দেখাবে:

```
┌──────────┬──────────┬──────────┐
│  Build   │   Test   │  Deploy  │
│    ✅    │    ✅    │    ✅    │
│  0.3s    │  0.2s    │  0.2s    │
└──────────┴──────────┴──────────┘
```

> **Stage View** দেখতে হলে Jenkins-এ **Pipeline Stage View Plugin** installed থাকতে হবে।

---
