## Jenkins File

#### Jenkins er New Item click koro--->Enter an item name: Declarative-Pipeline --->Select: Pipeline --->click: OK --->Pipeline e Script daw: 
```Script
pipeline{
    agent any
    
    stages{
        stage('Build'){
            steps{
                echo 'Building the project....'
            }
        }
        stage('Test'){
            steps{
                echo 'Running test....'
            }
        }
        stage('Deploy'){
            steps{
                echo 'Deploying the App....'
            }
        }
    }
}
```
---
![]()
