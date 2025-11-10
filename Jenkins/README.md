# Jenkins Master–Slave Setup — Submission

**Student:** Parth Nagar


**Date:** 2025-11-09

This repository demonstrates a distributed Jenkins setup with one controller (master) and one inbound agent (slave). The controller runs in Docker; the agent runs on Windows using `agent.jar` (WebSocket). A proof Pipeline executes on the agent.

---

## Contents

- `docker-compose.yml` — Spins up the Jenkins controller (LTS) on ports 8080/50000 with a persistent volume.
- `Jenkinsfile` — Minimal pipeline that runs on the agent and prints environment information.
- `report/Jenkins_Master_Agent_Report_Aashman.docx` — Word report with instructions and screenshot placeholders.
- `screenshots/` — Put all screenshots here before submitting/zip.

---

## How to Run (Controller in Docker)

1. **Create working folder** and save `docker-compose.yml` there.
2. Start Jenkins (Windows PowerShell):
   ```powershell
   docker compose up -d
   docker ps
   docker exec -it jenkins-master cat /var/jenkins_home/secrets/initialAdminPassword
   ```
3. Open `http://localhost:8080`, unlock with the password above, install suggested plugins, create admin user.

---

## Create the Agent in Jenkins UI

- Manage Jenkins → Nodes → New Node
  - Name: `agent1`
  - Type: Permanent Agent
  - # of executors: 1
  - Remote root directory: `/home/jenkins/agent` (just a label; Windows agent will use its own path)
  - Labels: `agent1`
  - Usage: Only build jobs with label expressions
  - Launch method: Launch agent by connecting it to the controller (inbound/JNLP)

Copy the **secret** from the agent page.

---

## Run the Agent on Windows (WebSocket)

Open PowerShell in a folder you choose (e.g. `D:\...\jenkins-lab\agent`) and run:

```powershell
curl.exe -sLo agent.jar http://localhost:8080/jnlpJars/agent.jar

java -jar .\agent.jar -url "http://localhost:8080/" `
  -secret "<PASTE_SECRET_HERE>" `
  -name "agent1" -webSocket `
  -workDir "D:\...\jenkins-lab\agent"
```

You should see logs: **WebSocket connection open**, **Connected**.
In Jenkins → Nodes, `agent1` goes **Online**.

---

## Pipeline Proof (runs on Windows agent)

A minimal `Jenkinsfile` (also in this repo) to prove execution on the agent:

```groovy
pipeline {
  agent { label 'agent1' }
  stages {
    stage('Environment') {
      steps {
        bat 'echo Running on %COMPUTERNAME% & whoami & cd'
      }
    }
    stage('Java Check') {
      steps {
        bat 'java -version'
      }
    }
  }
}
```

Create **New Item → Pipeline → demo-on-agent**, paste the script above, Save, **Build Now**, and capture the Console Output.


## Notes

- Controller is named `jenkins-master` in Docker.
- Agent connects via WebSocket; no port 50000 is required in this mode.
- Data persists in Docker volume `jenkins_home`.

---

