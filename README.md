#  Jenkins Pipeline with Slack Notifications 

##  Project Overview

This pipeline performs the following steps:

1. **Checkout Code** from GitHub
2. **Build Application** using Maven
3. **Deploy WAR** file to Tomcat
4. **Send Notifications to Slack** (Success / Failure / Unstable)

---

## 🛠️ Tech Stack

* Jenkins (CI/CD)
* Maven (Build Tool)
* Apache Tomcat (Deployment)
* Slack (Notifications via Webhook)

---

##  Slack Configuration (Webhook Method)

### Step 1: Create Slack App

* Go to: https://api.slack.com/apps
* Click **Create New App**
* Choose **From Scratch**

### Step 2: Enable Incoming Webhooks

* Navigate to **Incoming Webhooks**
* Activate it
* Click **Add New Webhook to Workspace**
* Select a channel (e.g., `#devops`)

### Step 3: Copy Webhook URL

Example:

```
https://hooks.slack.com/services/XXXX/XXXX/XXXX
```

---

## 🔐 Jenkins Configuration

### Add Credentials in Jenkins

* Go to:
  `Manage Jenkins → Manage Credentials`
* Add:

  * **Kind:** Secret Text
  * **Secret:** Slack Webhook URL
  * **ID:** `slack-webhook`

---

## ⚙️ Jenkins Pipeline

```groovy
pipeline {
    agent any

    stages {
    
        stage('checkout') {
            steps {
                echo 'git checkout stage'
                git branch: 'main', url: 'https://github.com/jadalaramani/slack_tomcat_jenkins.git'
            }
        }

        stage('build') {
            steps {
                echo 'Building with maven'
                sh 'mvn clean install'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying to tomcat'
                deploy adapters: [tomcat9(
                    credentialsId: 'tomcat',
                    url: 'http://13.217.10.33:8081/'
                )],
                contextPath: 'slack',
                war: '**/*.war'
            }
        }        
    }

    post {

        success {
            withCredentials([string(credentialsId: 'slack-webhook', variable: 'SLACK_URL')]) {
                sh """
                curl -X POST -H "Content-type: application/json" \
                --data "{\\"text\\":\\"✅ SUCCESS: ${JOB_NAME} Build #${BUILD_NUMBER} deployed successfully 🚀\\nURL: ${BUILD_URL}\\"}" \
                $SLACK_URL
                """
            }
        }

        failure {
            withCredentials([string(credentialsId: 'slack-webhook', variable: 'SLACK_URL')]) {
                sh """
                curl -X POST -H "Content-type: application/json" \
                --data "{\\"text\\":\\"❌ FAILURE: ${JOB_NAME} Build #${BUILD_NUMBER} failed 💥\\nURL: ${BUILD_URL}\\"}" \
                $SLACK_URL
                """
            }
        }
    }
}
```

---

##  Sample Slack Messages

* ✅ SUCCESS: Job deployed successfully
* ❌ FAILURE: Build failed
* 🔗 Includes Build URL for quick access

---

## ⚠️ Common Issues & Fixes

| Issue            | Cause         | Fix                    |
| ---------------- | ------------- | ---------------------- |
| Invalid JSON     | Wrong quoting | Escape quotes properly |
| curl error       | Bad URL       | Use correct webhook    |
| No message       | Wrong channel | Verify Slack webhook   |
| Pipeline failure | Syntax error  | Use `post` block       |

---

##  Learning Outcome

* Jenkins Pipeline automation
* Slack integration using Webhooks
* Secure credential handling in Jenkins
* Real-time CI/CD notifications

---

##  Author

**Jadala Ramani**
DevOps Engineer | AWS | Kubernetes | CI/CD
