
# 📘 Report: Hands-on #11 – AWS EC2, Spark & Docker Web App
**Submitted by:** Nikil Teja  
**Student Email:** sswarna@uncc.edu

---

## ✅ Objective

This assignment focused on:
- Running a PySpark word count on AWS EC2 with S3 integration.
- Creating and deploying a Dockerized Node.js web application.
- Learning cloud-based big data processing and container deployment.

---

## 🔹 TASK 1: Spark Word Count on EC2 + S3

### ⚙️ EC2 Setup (Amazon Linux 2)

```bash
ssh -i "word_key_pair.pem" ec2-user@<EC2_PUBLIC_IP>
```
🔹 *Connects to the EC2 instance using SSH and your private key.*

```bash
sudo yum update -y
```
🔹 *Updates all system packages to the latest versions.*

```bash
sudo yum install java-11 -y
```
🔹 *Installs Java 11 — required for Spark to run.*

```bash
export JAVA_HOME=$(dirname $(dirname $(readlink -f $(which java))))
```
🔹 *Sets the JAVA_HOME environment variable properly.*

```bash
java --version
```
🔹 *Verifies that Java is installed and accessible.*

```bash
sudo mount -o remount,size=2G /tmp
```
🔹 *Increases `/tmp` directory size to avoid Spark temporary storage issues.*

```bash
sudo yum install python3-pip -y
pip3 install pyspark
```
🔹 *Installs pip and then installs PySpark (the Python interface for Apache Spark).*

---

### ⚙️ PySpark Word Count Script

**File: `word_count.py`**

```python
from pyspark.sql import SparkSession

# AWS Credentials
AWS_ACCESS_KEY_ID = 'YOUR_ACCESS_KEY' 
AWS_SECRET_ACCESS_KEY = 'YOUR_SECRET_KEY' 

# S3 paths
S3_INPUT = 's3a://nikilteja/word_count_nikil.txt'
S3_OUTPUT = 's3a://nikilteja/output_folder/'

# Spark Session
spark = SparkSession.builder \
    .appName("WordCount") \
    .config("spark.jars.packages", "org.apache.hadoop:hadoop-aws:3.3.1,com.amazonaws:aws-java-sdk-bundle:1.11.901") \
    .getOrCreate()

# Hadoop S3 Configuration
hadoop_conf = spark.sparkContext._jsc.hadoopConfiguration()
hadoop_conf.set("fs.s3a.access.key", AWS_ACCESS_KEY_ID)
hadoop_conf.set("fs.s3a.secret.key", AWS_SECRET_ACCESS_KEY)

# Word Count Logic
text_file = spark.sparkContext.textFile(S3_INPUT)
counts = text_file.flatMap(lambda line: line.split()) \
                  .map(lambda word: (word, 1)) \
                  .reduceByKey(lambda a, b: a + b)
counts.saveAsTextFile(S3_OUTPUT)
spark.stop()
```

🔹 *This script reads a text file from S3, performs word count using PySpark, and writes the output back to S3.*

---

## 🔹 TASK 2: Dockerized Node.js Web App

### 📁 Create Node.js App

```bash
mkdir node-webserver && cd node-webserver
```
🔹 *Creates and enters the project folder.*

```bash
npm init -y
```
🔹 *Initializes a Node.js project with default settings.*

```bash
npm install express
```
🔹 *Installs the Express web framework.*

---

**File: `server.js`**

```js
const express = require('express');
const app = express();
const port = 3000;

app.get('/', (req, res) => {
    res.send('Hello, World! Running in Docker.');
});

app.listen(port, () => console.log(`Server running at http://localhost:${port}`));
```

🔹 *Sets up a basic Express server that responds to HTTP GET requests.*

---

### 🐳 Dockerfile

```Dockerfile
FROM node:20               # changed from node:14 → node:20
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

🔹 *Defines the Docker image:*
- Uses Node.js 20 instead of 14
- Installs dependencies
- Starts the server

---

### 🔧 Docker on EC2

```bash
sudo yum install docker -y
```
🔹 *Installs Docker.*

```bash
sudo service docker start
```
🔹 *Starts Docker service.*

```bash
sudo usermod -aG docker ec2-user
```
🔹 *Adds the current user to the Docker group.*

```bash
logout  # then re-login for group changes to apply
```

---

### 🛠️ Build & Run Container

```bash
docker build -t nikilteja215/webserver:latest .
```
🔹 *Builds the Docker image.*

```bash
docker run -d -p 80:3000 nikilteja215/webserver:latest
```
🔹 *Runs the container and maps port 80 of EC2 to 3000 inside the container.*

---

### 🌐 Access App

```bash
curl http://localhost
```
🔹 *Checks if the app is running locally.*

```bash
curl http://checkip.amazonaws.com
```
🔹 *Gets your EC2’s public IP to test in browser.*

Access in browser:  
**http://<EC2_PUBLIC_IP>**  
✔️ *App says: "Hello, World! Running in Docker."*

---

### 📤 DockerHub Push

```bash
docker login
```
🔹 *Logs into DockerHub.*

```bash
docker tag nikilteja215/webserver:latest nikilteja215/webserver:latest
docker push nikilteja215/webserver:latest
```

## 🔧 Troubleshooting Done

- ❌ `Output directory already exists` → deleted S3 output folder
- 🛑 GitHub blocking push due to secrets → removed AWS keys and used `git filter-repo`
- ⚠️ Merge error (`unrelated histories`) → fixed with `--allow-unrelated-histories`
- ✉️ Email privacy error → used GitHub noreply email

---

## 🔗 Links

- **GitHub Repo:** [https://github.com/nikilteja215/word-count-spark/tree/main](https://github.com/nikilteja215/word-count-spark/tree/main)
- **DockerHub Repo:** [https://hub.docker.com/repository/docker/nikilteja215/webserver/general](https://hub.docker.com/repository/docker/nikilteja215/webserver/general)
- **Web App (EC2):** http://3.237.83.253

---

## ✅ Submission Checklist

- [x] EC2 instance created
- [x] PySpark word count completed
- [x] Docker container built and deployed
- [x] DockerHub image pushed
- [x] GitHub repo cleaned and submitted
- [x] Final report written

---

## 🙌 Done By

**Nikil Teja**  
UNCC Student  
Email: sswarna@uncc.edu  
GitHub: [@nikilteja215](https://github.com/nikilteja215)
