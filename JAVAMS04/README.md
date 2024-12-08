# Working with Cloud Trace

## Table of Contents
1. [Fetch the Application Source Files](#1-fetch-the-application-source-files)
2. [Enable Cloud Trace API](#2-enable-cloud-trace-api)
3. [Add the Spring Cloud GCP Trace Starter](#3-add-the-spring-cloud-gcp-trace-starter)
4. [Configure Applications](#4-configure-applications)
5. [Set Up a Service Account](#5-set-up-a-service-account)
6. [Run the Application Locally with Your Service Account](#6-run-the-application-locally-with-your-service-account)
7. [Examine the Trace](#7-examine-the-trace)

---

## 1. Fetch the Application Source Files

### Set the Project ID Environment Variable
```bash
export PROJECT_ID=$(gcloud config list --format 'value(core.project)')
```

### Verify and Copy Application Files
```bash
gcloud storage ls gs://$PROJECT_ID
gcloud storage cp -r gs://$PROJECT_ID/* ~/
```

### Make Maven Wrapper Scripts Executable
```bash
chmod +x ~/guestbook-frontend/mvnw
chmod +x ~/guestbook-service/mvnw
```

---

## 2. Enable Cloud Trace API
```bash
gcloud services enable cloudtrace.googleapis.com
```

---

## 3. Add the Spring Cloud GCP Trace Starter

Insert the following dependency into `~/guestbook-service/pom.xml` and `~/guestbook-frontend/pom.xml`:
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-gcp-starter-trace</artifactId>
</dependency>
```

---

## 4. Configure Applications

### Update Application Properties

In `guestbook-service/src/main/resources/application.properties` and `guestbook-frontend/src/main/resources/application.properties`, add:
```properties
spring.cloud.gcp.trace.enabled=false
```

In `guestbook-service/src/main/resources/application-cloud.properties` and `guestbook-frontend/src/main/resources/application-cloud.properties`, add:
```properties
spring.cloud.gcp.trace.enabled=true
spring.sleuth.sampler.probability=1.0
spring.sleuth.scheduled.enabled=false
```

---

## 5. Set Up a Service Account

### Create a Service Account
```bash
gcloud iam service-accounts create guestbook
```

### Grant Permissions
```bash
export PROJECT_ID=$(gcloud config list --format 'value(core.project)')
gcloud projects add-iam-policy-binding ${PROJECT_ID} \
  --member serviceAccount:guestbook@${PROJECT_ID}.iam.gserviceaccount.com \
  --role roles/editor
```
> **Note:** In production, assign only the roles and permissions that the application needs.

### Create a Service Account Key
```bash
gcloud iam service-accounts keys create \
    ~/service-account.json \
    --iam-account guestbook@${PROJECT_ID}.iam.gserviceaccount.com
```
> **Note:** Treat the `service-account.json` file as confidential. Do not share it.

---

## 6. Run the Application Locally with Your Service Account

### Backend Service
```bash
cd ~/guestbook-service

./mvnw spring-boot:run \
  -Dspring-boot.run.jvmArguments="-Dspring.profiles.active=cloud \
  -Dspring.cloud.gcp.credentials.location=file:///$HOME/service-account.json"
```

### Frontend Service
```bash
cd ~/guestbook-frontend

./mvnw spring-boot:run \
  -Dspring-boot.run.jvmArguments="-Dspring.profiles.active=cloud \
  -Dspring.cloud.gcp.credentials.location=file:///$HOME/service-account.json"
```

---

## 7. Examine the Trace

- Open the Google Cloud console in your browser.
- Search for **Trace** in the console search bar and click the **Trace** result.
- Set the time range to **1 hour** and enable **Auto reload**.
- Wait for new trace data (may take up to 30 seconds).
- Click the blue dot to view trace details.
