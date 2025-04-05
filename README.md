
# Server setup with Jenkins

## Server & Network Configuration

Configure public ports and inbound rules via Azure or AWS configuration portal.

### Example Ports
| Port | Purpose         | Inbound ID | Protocol |
|------|------------------|------------|----------|
| 22   | SSH              | Admin IP       | TCP      |
| 8000 | Backend (API)    | Public       | TCP      |
| 8001 | Frontend (Web)   | Public       | TCP      |
| 5432 | PostgreSQL       | Dev Ip       | TCP      |
| 9000 | MinIO API        | Dev Ip       | TCP      |
| 9001 | MinIO Console    | Dev Ip       | TCP      |
| 8080 | Jenkins          | Admin IP       | TCP      |

---

## SSH & Server Setup

### Install Docker

```bash
sudo apt update
sudo apt upgrade -y

sudo apt install ca-certificates curl gnupg lsb-release -y
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

# Verify installation
docker --version
```

---

### Install Jenkins

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install openjdk-17-jdk -y
```

```bash
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update
sudo apt install jenkins -y

sudo systemctl start jenkins
sudo systemctl enable jenkins
```

---

## Docker Compose Setup

### Example `docker-compose.yml`

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15
    container_name: my_postgres
    restart: always
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: mydb
    ports:
      - "5432:5432"
    volumes:
      - pg_data:/var/lib/postgresql/data
    networks:
      - my_network

  minio:
    image: quay.io/minio/minio
    container_name: my_minio
    command: server /data --console-address ":9001"
    restart: always
    ports:
      - "9000:9000"
      - "9001:9001"
    environment:
      MINIO_ROOT_USER: minioadmin
      MINIO_ROOT_PASSWORD: miniosecret
    volumes:
      - minio_data:/data
    networks:
      - my_network

volumes:
  pg_data:
  minio_data:

networks:
  my_network:
    driver: bridge
```

### Steps to Run

1. Create file:
   ```bash
   nano docker-compose.yml
   ```

2. Paste the content above and save.

3. Start services:
   ```bash
   sudo docker compose up -d
   ```

4. Verify:
   ```bash
   sudo docker ps -a
   ```

---

## Jenkins Usage

### Concept

- Jenkins Pipeline is used for automating CI/CD.
- It is triggered either by Git webhook or manual trigger.
- Jenkins uses the `Jenkinsfile` stored in the repository.

#### Basic Pipeline Steps:
- Pull source code from the repository
- Build Docker image using the repository’s `Dockerfile`
- Remove old container and run the updated one

---

### Access Jenkins

- URL: `http://<your-server-ip>:8080`
- Default credentials:
  - **Username:** `admin`
  - **Password:** 
    ```bash
    sudo cat /var/lib/jenkins/secrets/initialAdminPassword
    ```

---

### Creating Jenkins Credentials

Navigate to:  
**Manage Jenkins → Credentials → Create Credentials**

Use this to securely store:
- GitHub Personal Access Token (instead of a password)
- Database credentials
- DockerHub credentials (if pushing images)

> ⚠️ GitHub requires 2FA. Do **not** use your GitHub password — use a **Personal Access Token (PAT)** instead.

---

### Creating a Pipeline

1. Go to **New Item**
2. Enter a name → Select **Pipeline** → Click **OK**
3. Choose **Pipeline from SCM**
4. Ensure your repo contains both `Dockerfile` and `Jenkinsfile`
5. Can use sample Dockerfile and Jenkinsfile and replace custom information.
