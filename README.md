# 🧪 Postman Platzi Fake Store  API Automation Framework (Newman + Jenkins + Allure)

A fully automated **Platzi Fake Store  API testing framework** built using **Postman**, **Newman CLI**, and **Jenkins CI/CD** — featuring data-driven testing, environment-driven executions, JSON schema validations, and dual HTML + Allure reporting.

---

## 📘 Overview

This project demonstrates an **end-to-end Platzi Fake Store API automation pipeline** that runs 50+ Postman test cases (both positive and negative) across multiple collections, integrated with **Newman** for CLI execution and **Jenkins** for CI automation.

The suite includes:
- ✅ Test chaining (endpoints depend on previous responses)
- ⚙️ Dynamic pre-request & post-request scripts
- 🧩 JSON Schema validation
- 📂 CSV-driven data parameterization
- 📈 Integrated Allure + Newman HTML reports in Jenkins
- 🌐 Environment management inside Jenkins
- 🧾 100% functional CI pipeline for API quality gates

---

## 🏗️ Project Structure
E:\Postman Project
│
├── Collections
│ ├── Positive test cases.postman_collection.json
│ ├── Negative test cases.postman_collection.json
│ |
│ └── QA.postman_environment.json # Environment for all collections
│
├── Csv files
│ ├── Create_a_product.csv
│ ├── Create_a_user.csv
│ └── ... # Other data files
├── newman-reports\ # Generated HTML reports
│
├── allure-results\ # Raw Allure JSON + attachments
├── allure-report\ # Generated Allure HTML report
│
├── Jenkinsfile # Pipeline script
├── run_newman.bat # Local Windows runner (manual mode)

Run a Single Collection
newman run "E:\Postman Project\Collections\Positive test cases.postman_collection.json" ^
  -e "E:\Postman Project\Collections\QA.postman_environment.json" ^
  --reporters cli,htmlextra ^
  --reporter-htmlextra-export "E:\Postman Project\newman-reports\Positive-tests.html"
  
Run Data-Driven Collection
newman run "E:\Postman Project\Collections\Negative test cases.postman_collection.json" ^
  -e "E:\Postman Project\Collections\QA.postman_environment.json" ^
  -d "E:\Postman Project\Csv files\Create_a_user.csv" ^
  --folder "Users unhappy path" ^
  --reporters cli,htmlextra,allure

🧩 Jenkins Pipeline Overview
| Stage                | Description                                                        |
| -------------------- | ------------------------------------------------------------------ |
| **Prep & Cleanup**   | Verifies Newman CLI, deletes previous Allure reports               |
| **Run API Tests**    | Executes collections & CSV-driven runs |
| **Generate Reports** | Generates and publishes Allure + HTMLExtra reports                 |
| **Publish Results**  | Jenkins auto-publishes both reports under build artifacts          |

📄 Jenkinsfile Key Snippet
pipeline {
  agent any
  options { timestamps() disableConcurrentBuilds() }

  environment {
    COLL_DIR = 'E:\\Postman Project\\Collections'
    CSV_DIR  = 'E:\\Postman Project\\Csv files'
    ENV_NAME = 'QA.postman_environment.json'
    NEWMAN_BIN = 'C:\\Users\\raxit\\AppData\\Roaming\\npm\\newman.cmd'
    ALLURE_RESULTS = 'allure-results'
    ALLURE_REPORT  = 'allure-report'
  }

  stages {
    stage('Prep & Cleanup') {
      steps {
        bat '''
          del /q "%ALLURE_RESULTS%" "%ALLURE_REPORT%"
          call "%NEWMAN_BIN%" -v
        '''
      }
    }
    stage('Run Tests') {
      steps {
        bat '''
          call "%NEWMAN_BIN%" run "%COLL_DIR%\\Positive test cases.postman_collection.json" ^
          -e "%COLL_DIR%\\%ENV_NAME%" ^
          --reporters cli,htmlextra,allure
        '''
      }
    }
    stage('Generate Reports') {
      steps {
        bat 'allure generate "%ALLURE_RESULTS%" --clean -o "%ALLURE_REPORT%"'
      }
    }
  }

  post {
    always {
      publishHTML([reportDir: 'newman-reports', reportFiles: 'Positive-tests.html', reportName: 'HTML Report'])
      allure includeProperties: false, jdk: '', results: [[path: 'allure-results']]
    }
  }
}

📊 Reporting
🧭 Allure Report

Full request/response history

Screenshots, attachments, environment labels

Build trend history & graphs

Severity + owner + environment tags

Access:
Jenkins → Build → Allure Report

💡 Newman HTMLExtra Report

High-level summary of each request

Response time, status, and assertion logs

Failure breakdown by endpoint

Access:
Jenkins → Build → HTML Report

🧠 Data & Environment Design
| File                          | Purpose                                      |
| ----------------------------- | -------------------------------------------- |
| `QA.postman_environment.json` | Holds base URLs, auth tokens, and env vars   |
| `Create_a_product.csv`        | Product test data (positive & negative)      |
| `Create_a_user.csv`           | Negative scenarios for user creation         |

🔐 Credentials Handling

API keys & tokens are stored as Jenkins credentials (not hardcoded)

Injected at runtime as environment variables

Sensitive headers & tokens are redacted in console output


🏁 Summary

This project showcases a real-world grade Postman + Jenkins pipeline capable of:

Chained, data-driven API testing

Pre/post script automation

Dual report generation

CI/CD integration

Environment isolation

Build-quality enforcement

Perfect for demonstrating API automation maturity in both functional and CI pipelines.

