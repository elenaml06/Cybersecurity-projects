# Mimikatz Detection - Log Analysis

This project demonstrates how to analyze event logs using **ElasticSearch** and **Kibana** to detect potential threats, specifically focusing on the detection of **Mimikatz** activity. The goal of this project is to create a logging and detection framework that identifies security events, correlates them, and provides insights into possible credential dumping attacks or lateral movement techniques.

## Project Overview

In this project, we used **Docker** to deploy the **Elastic Stack (ElasticSearch & Kibana)**. After setting up the environment, we imported a set of logs from a **ZIP file** (downloaded from a third-party source). We then used **Kibana** to search and analyze the logs for any suspicious activity, including detecting **Mimikatz** activity and potential credential dumping.

### Key Features:
- Use of Docker to deploy the Elastic Stack (ElasticSearch & Kibana)
- Import and analyze logs from a third-party source (ZIP file)
- Detection of **Mimikatz** activity and other credential dumping techniques
- Visualize and query logs using **Kibana** dashboards

## Technologies Used

- **Docker**: For deploying ElasticSearch and Kibana
- **ElasticSearch**: For storing and indexing logs
- **Kibana**: For visualizing logs and querying data
- **Logs**: Downloaded from a third-party source (ZIP file)
- **Mimikatz**: A tool used in credential dumping attacks

## Setup and Installation

### Step 1: Install Docker
```
   sudo apt update
   sudo apt install docker.io
   sudo systemctl enable --now docker
   docker --version
```
create docker-compose.yml
```
   nano docker-compose.yml

version: '3.7'

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.10.0
    container_name: elasticsearch
    environment:
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ports:
      - "9200:9200"
      - "9300:9300"
    networks:
      - elastic_network
    volumes:
      - elasticsearch_data:/usr/share/elasticsearch/data

  kibana:
    image: docker.elastic.co/kibana/kibana:7.10.0
    container_name: kibana
    networks:
      - elastic_network
    ports:
      - "5601:5601"
    depends_on:
      - elasticsearch

networks:
  elastic_network:
    driver: bridge

volumes:
  elasticsearch_data:
    driver: local
```
execute docker, add your user, and run Elasticsearch
```
   sudo docker-compose up -d
   sudo usermod -aG docker $USER
   docker run --name elasticsearch -d -p 9200:9200 -p 9300:9300 docker.elastic.co/elasticsearch/elasticsearch:7.10.0
```
Check if it appears in the container
```
   docker ps
```
Open http://localhost:5601 in your browser to start using Elasticsearch

## Step 4: Download and Import Logs into ElasticSearch

For this project, we used logs that were downloaded from a third-party source. These logs were not Windows Event Logs but were provided as a ZIP file containing sample logs.

1. **Download the Logs**  
   Download the logs from a third-party source. You can use the link provided below:  
   **https://github.com/OTRF/Security-Datasets/blob/master/datasets/atomic/windows/credential_access/host/empire_mimikatz_lsadump_inject_protected_mode.zip**

2. **Extract the ZIP File**  
   After downloading the ZIP file, extract it to a folder on your local machine.

3. **Upload Logs to ElasticSearch**  
   To upload the logs into **ElasticSearch**, you can either use the **Kibana UI** or a log shipper like **Filebeat**. For this guide, we'll upload the logs manually via Kibana.

### Using the Kibana UI to Import Logs

1. **Access Kibana**:  
   Go to the **Kibana UI** at `http://localhost:5601`.

2. **Create an Index Pattern**:  
   In the **Kibana** menu, navigate to **Management** > **Index Patterns** > **Create Index Pattern**.

3. **Select the Index Pattern**:  
   Select the appropriate index pattern for your logs (e.g., `logstash-*` or any other pattern that matches the structure of the logs you want to analyze). You can also specify custom index patterns based on the format of your logs.

4. **Ingest the Logs**:  
   After selecting the index pattern, **Kibana** will start ingesting the logs into **ElasticSearch**. This may take a few minutes depending on the size of the logs.

5. **Start Analyzing the Logs**:  
   Once the logs are ingested, you can start analyzing them. Go to the **Discover** tab in **Kibana** to search and filter through the logs. You can also use **Kibana Visualizations** to create dashboards for better insights.

---

## Step 5: Analyze Logs for Suspicious Activity

After importing the logs into **ElasticSearch** and setting up the index pattern in **Kibana**, I began an in-depth analysis to identify suspicious activity, particularly focusing on potential **Mimikatz** usage, credential dumping, and lateral movement techniques. The following outlines the process I followed to identify indicators of compromise (IoC) related to **Mimikatz**, **LSASS**, and other suspicious behavior.

### Initial Search for Known Indicators of Mimikatz and Credential Dumping

The first step in the analysis was to search for specific keywords and indicators that are commonly associated with **Mimikatz** and **credential dumping**. These include known Mimikatz commands and behaviors that typically appear in Windows Event Logs during an attack. I started with the following terms:

- **Mimikatz** (e.g., `sekurlsa`, `lsadump`, `kerberos`, `mimikatz`)
- **LSASS** (Local Security Authority Subsystem Service, commonly targeted for credential dumping)
- **DLL Files** (often loaded by Mimikatz during its execution)

### 🧩 Key Indicators of Mimikatz Activity

- **Sysmon Event ID 10** – *Process Access*:  
  Shows `svchost.exe` accessing `mimikatz.exe`, indicating possible inter-process interaction or injection.

- **Security Event ID 4656** – *Handle to LSASS Requested*:  
  `mimikatz.exe` attempts to access `lsass.exe`, requesting permissions to:
  - Read process memory  
  - Write to process memory  
  - Create threads  
  - Query information  

  This behavior matches Mimikatz attempts to dump credentials from memory.
  🛑 **This access was denied**, as indicated by the `AUDIT_FAILURE`. While the attempt failed, this is still a high-confidence indicator of malicious behavior, as it matches Mimikatz's known signature for credential dumping attempts.

- **Process Name and Path**:  
  The binary is clearly labeled:  
  `C:\Users\pgustavo\Downloads\mimikatz_trunk\x64\mimikatz.exe`

### 🧠 Insights

These logs provide enough data to:
- Confirm **Mimikatz was executed**
- Correlate its interaction with **LSASS**
- Map permissions requested that match known **credential dumping techniques**

Although these logs don’t show the final outcome (e.g., extracted credentials), the activity strongly suggests an **attempted or successful credential dump**.

### 🔍 Other Search Attempts (No Relevant Findings)

During the investigation, the following techniques and methods were searched for, but no relevant data was found:

1. **Sekurlsa**:  
   Attempts to identify any interactions with the `sekurlsa` module or its associated functions, commonly used by Mimikatz for dumping credentials, were unsuccessful.

2. **DCsync**:  
   No traces were found of the `DCsync` technique, which is often used to replicate domain controller credentials using the MS-RPC protocol.

3. **Lsadump**:  
   No logs were found related to the `lsadump` technique, which is used by Mimikatz to extract information from the LSASS process, particularly password hashes.

4. **PowerShell**:  
   No evidence of PowerShell-based credential dumping was found, which is another common method for dumping credentials in Mimikatz.

5. **Reg Save**:  
   No instances of registry hive dumping using `reg save` were detected, which can sometimes be used for credential extraction or system information collection.

6. **LogonPasswords**:  
   No entries or references to the `logonpasswords` plugin, typically used in Mimikatz for extracting plaintext passwords from memory, were found.

7. **NTDS.dit**:  
   No evidence was found indicating the extraction of the NTDS.dit file, which stores Active Directory credentials. This file is often targeted by attackers using Mimikatz to dump credentials.

### 🛡️ Detection Strategy

Security teams should:
- Monitor for access attempts on `lsass.exe` and similar critical processes.
- Look for unusual processes interacting with sensitive system processes like `lsass.exe`.
- Set up alerts for Event IDs:  
  - **10** (Sysmon: Process Access)  
  - **4656** (Security Auditing: Handle to Object Requested)






https://github.com/OTRF/Security-Datasets/blob/master/datasets/atomic/windows/credential_access/host/empire_mimikatz_lsadump_inject_protected_mode.zip
