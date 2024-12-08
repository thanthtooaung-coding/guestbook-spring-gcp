# Configuring and Connecting to Cloud SQL

## Table of Contents
1. [Fetch the Application Source Files](#1-fetch-the-application-source-files)
2. [Create a Cloud SQL Instance, Database, and Table](#2-create-a-cloud-sql-instance-database-and-table)
3. [Add Cloud SQL Support to Your Application](#3-add-cloud-sql-support-to-your-application)
4. [Configure an Application Profile for Cloud SQL](#4-configure-an-application-profile-for-cloud-sql)

---

## 1. Fetch the Application Source Files

```bash
git clone --depth=1 https://github.com/GoogleCloudPlatform/training-data-analyst
ln -s ~/training-data-analyst/courses/java-microservices/spring-cloud-gcp-guestbook ~/spring-cloud-gcp-guestbook

cp -a ~/spring-cloud-gcp-guestbook/1-bootstrap/guestbook-service ~/guestbook-service
cp -a ~/spring-cloud-gcp-guestbook/1-bootstrap/guestbook-frontend ~/guestbook-frontend
```

---

## 2. Create a Cloud SQL Instance, Database, and Table

### Enable the Cloud SQL Administration API
```bash
gcloud services enable sqladmin.googleapis.com
```

### Confirm the API is Enabled
```bash
gcloud services list | grep sqladmin
```

### Create and Configure Cloud SQL
1. List existing Cloud SQL instances:
    ```bash
    gcloud sql instances list
    ```
2. Create a new Cloud SQL instance:
    ```bash
    gcloud sql instances create guestbook --region=us-central1
    ```
3. Create a database in the instance:
    ```bash
    gcloud sql databases create messages --instance guestbook
    ```
4. Connect to the Cloud SQL instance and create the schema:
    ```bash
    gcloud sql connect guestbook
    
    # SQL commands:
    show databases;
    use messages;

    CREATE TABLE guestbook_message (
      id BIGINT NOT NULL AUTO_INCREMENT,
      name CHAR(128) NOT NULL,
      message CHAR(255),
      image_uri CHAR(255),
      PRIMARY KEY (id)
    );

    exit
    ```

---

## 3. Add Cloud SQL Support to Your Application

### Add the Dependency
Add the following to your `pom.xml`:
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-gcp-starter-sql-mysql</artifactId>
</dependency>
```

### Update Application Properties
Open `guestbook-service/src/main/resources/application.properties` and add:
```properties
spring.cloud.gcp.sql.enabled=false
```

---

## 4. Configure an Application Profile for Cloud SQL

### Find the Instance Connection Name
```bash
gcloud sql instances describe guestbook --format='value(connectionName)'
```

### Create Application Cloud Properties
Open `guestbook-service/src/main/resources/application-cloud.properties` and add:
```properties
spring.cloud.gcp.sql.enabled=true
spring.cloud.gcp.sql.database-name=messages
spring.cloud.gcp.sql.instance-connection-name=YOUR_INSTANCE_CONNECTION_NAME
spring.datasource.hikari.maximum-pool-size=5
```

---

## Test the Backend Service Running on Cloud SQL

### Run Tests
```bash
cd ~/guestbook-service
./mvnw test
```

### Start the Service
```bash
./mvnw spring-boot:run \
  -Dspring-boot.run.jvmArguments="-Dspring.profiles.active=cloud"
```

### Test the API Endpoints

1. Add a message:
    ```bash
    curl -XPOST -H "content-type: application/json" \
      -d '{"name": "Ray", "message": "Hello CloudSQL"}' \
      http://localhost:8081/guestbookMessages
    ```
2. Retrieve messages:
    ```bash
    curl http://localhost:8081/guestbookMessages
    ```
3. Verify the data in the database:
    ```bash
    gcloud sql connect guestbook

    # SQL commands:
    use messages;
    select * from guestbook_message;

    exit
    ```

### Expected Output:
```plaintext
+----+------+----------------+-----------+
| id | name | message        | image_uri |
+----+------+----------------+-----------+
|  1 | Ray  | Hello CloudSQL | NULL      |
+----+------+----------------+-----------+
1 row in set (0.04 sec)
