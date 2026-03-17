# 🔥 Big Data with Apache Spark, HDFS, Docker & Kubernetes

This repository contains code, datasets, and execution guides for **Apache Spark** workloads using **RDDs**, **DataFrames**, and **Map/Reduce**, both on a **local Docker-based setup** and on a **distributed Kubernetes environment**, for the course [Big Data Management](https://dsml.ece.ntua.gr/studies/courses/diacheirise-dedomenon-megales-klimakas) of the [Interdepartmental MSc Program in Data Science and Machine Learning](https://dsml.ece.ntua.gr/) of the [National Technical University of Athens](http://www.ntua.gr).

---

## 📘 Recommended study / execution order

1. [00_Preparatory-lab](docs/00_Preparatory-lab): Environment preparation (WSL + Docker Desktop)
2. [00_pycharm](docs/00_pycharm): Running Spark locally with PyCharm
3. [01_lab1-docker](docs/01_lab1-docker): Launching Spark + HDFS with Docker Compose
4. [01_lab1-k8s](docs/01_lab1-k8s): Running Spark jobs on Kubernetes (CSLAB)
5. [02_lab2](docs/02_lab2): Join queries using RDDs and DataFrames

📁 The same guides are also available under [`odigoi/`](./odigoi) in English `.docx` format, alongside the original Greek handouts.

The Word guides under `odigoi/` are generated from the Markdown files in `docs/` with Pandoc.
On Windows, `scripts/export-docx.ps1` and `make -C docs docx` also use Microsoft Word to refresh the table of contents after export.

---

## 📁 Repository structure

- `code/`: Spark code in Python (RDD and DataFrame examples)
- `examples/`: Sample CSV / text files (`employees`, `departments`, `text`)
- `docker/`
  - `01-lab1-spark-hdfs/`: Spark + HDFS setup with Docker Compose
  - `02-lab2-spark-history-server/`: Spark History Server setup with Docker
- `docs/`: Guides in Markdown format
- `odigoi/`: Handouts in `.docx` and `.pdf`

---

## 💻 Running Spark with PyCharm (local development)

📄 Guide: [`00_pycharm`](docs/00_pycharm) · [English Markdown](docs/00_pycharm/README.en.md) · [English DOCX](odigoi/0_pycharm_spark_implementation.en.docx)

- Use `venv`, install `pyspark` and `psutil`
- Configure the required environment variables in the Run Configuration
- Access the Spark UI at `localhost:4040`

---

## 🧱 Environment preparation (WSL + Docker Desktop)

📄 Guide: [`00_Preparatory-lab`](docs/00_Preparatory-lab) · [English Markdown](docs/00_Preparatory-lab/README.en.md) · [English DOCX](odigoi/0_Preparatory_lab_Docker_Desktop-wsl.en.docx)

- Install WSL 2 and Ubuntu
- Configure Docker Desktop to use the WSL backend
- Verify the installation with `hello-world`

---

## 🐳 Lab 01a: Running Spark + HDFS with Docker

📄 Guide: [`01_lab1-docker`](docs/01_lab1-docker) · [English Markdown](docs/01_lab1-docker/README.en.md) · [English DOCX](odigoi/01_lab1-docker.en.docx)

This lab sets up a local multi-container environment that includes:
- an HDFS cluster with one NameNode and three DataNodes
- a Spark cluster with one Master and three Workers
- persistent Docker volumes for uploaded code and data

Example command:

```bash
cd ~/bigdata-dsml/docker/01-lab1-spark-hdfs
docker-compose up --build -d
```

Upload example data to HDFS:

```bash
docker exec namenode hdfs dfs -put -f /mnt/upload/text.txt /user/root/text.txt
```

---

## ☁️ Lab 01b: Spark on Kubernetes

📄 Guide: [`01_lab1-k8s`](docs/01_lab1-k8s) · [English Markdown](docs/01_lab1-k8s/README.en.md) · [English DOCX](odigoi/01_lab1-k8s.en.docx)

- Connects to the lab infrastructure through OpenVPN
- Uses `kubectl` and `k9s` to manage and monitor execution
- Runs Spark jobs on the Kubernetes cluster while reading / writing data from HDFS

Example `spark-submit`:

```bash
spark-submit     --master k8s://https://termi7.cslab.ece.ntua.gr:6443     --deploy-mode cluster     --name wordcount     --conf spark.kubernetes.namespace=testuser-priv     --conf spark.executor.instances=5     --conf spark.eventLog.enabled=true     --conf spark.eventLog.dir=hdfs://hdfs-namenode:9000/user/testuser/logs     hdfs://hdfs-namenode:9000/user/testuser/wordcount_localdir.py
```

---

## 🔁 Lab 02: Join queries with RDDs and DataFrames

📄 Guide: [`02_lab2`](docs/02_lab2) · [English Markdown](docs/02_lab2/README.en.md) · [English DOCX](odigoi/02_lab2.en.docx)

This lab implements classic relational-style processing tasks in Spark, including:
- sorting and filtering employees
- joins between employees and departments
- yearly income calculation
- additional examples with the DataFrame API

It also includes setup instructions for the **Spark History Server** to inspect past executions via the web UI.

---

## ⚙️ Preparing data in HDFS

```bash
# Copy the example data and code into HDFS
hadoop fs -put examples examples
hadoop fs -put code code

# Verify
hadoop fs -ls examples
hadoop fs -ls code
```

---

## 🧪 Running Spark queries

| Query | Description | Implementation |
|---|---|---|
| Query 1 | 5 employees with the lowest salary | RDD / DF |
| Query 2 | 3 highest-paid employees in "Dep A" / salary sum per department | RDD / DF |
| Query 3 | Yearly income of all employees | RDD / DF |
| Query 4 | Join employees with departments using only RDDs | RDD |
| Word Count | Count word occurrences in a text file | RDD |

📈 The above queries can be inspected through the Spark History Server (see the end of [02_lab2](docs/02_lab2)).

Example:

```bash
# ⚠️ Replace ikons with your own username
spark-submit hdfs://hdfs-namenode:9000/user/ikons/code/RddQ1.py
```

---

## 👤 Maintainer

**Ioannis Konstantinou**

📬 Questions / issues: [GitHub Issues](https://github.com/ikons/bigdata-dsml/issues)
