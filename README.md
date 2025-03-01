# Integrating OllamaDeepSeek with n8n on a VM (Google Cloud)

This tutorial will guide you step-by-step to deploy **Ollama** on a virtual machine (VM) using Docker and connect it with **n8n** to automate workflows.

> **Requirements:** Ensure your VM has at least **4 GB of RAM**. If the VM does not have enough memory, the download of the **deepseek-coder** model will fail.

---

## 1. Prerequisites

- Docker installed on your VM.
- n8n deployed using Docker.
- Access to your VM's terminal.

### Add Your User to the Docker Group

To run Docker commands without `sudo`, add your user to the Docker group:

```bash
sudo usermod -aG docker $USER
```

> **Note:** After adding the user to the Docker group, **log out and log back in** for the changes to take effect.

### Set Permissions for the n8n Volume

Ensure the correct permissions are set for the n8n data volume:

```bash
sudo chown -R 1000:1000 /var/lib/docker/volumes/n8n_data/_data
sudo chmod -R 700 /var/lib/docker/volumes/n8n_data/_data
```

This grants the appropriate permissions for n8n to access its data.

---

## 2. Download the Ollama Image

Run the following command to download the official **Ollama** image from Docker Hub.

```bash
docker pull ollama/ollama
```

This command downloads the latest version of **Ollama**.

---

## 3. Run the Ollama Container

Now run the **Ollama** container and expose port `11434`.

```bash
docker run -d -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama
```

**Explanation:**
- `-d`: Runs the container in detached mode.
- `-v ollama:/root/.ollama`: Creates a persistent volume for the downloaded models.
- `-p 11434:11434`: Maps port `11434` of the container to the VM.
- `--name ollama`: Names the container **ollama**.

---

## 4. Create a Docker Network for Container Communication

Create a custom Docker network named `n8n_network`.

```bash
docker network create n8n_network
```

This allows **n8n** and **Ollama** to communicate with each other.

---

## 5. Connect the Containers to the Custom Network

Connect both **n8n** and **Ollama** to the `n8n_network`.

### Connect n8n to the network:

```bash
docker network connect n8n_network n8n
```

### Connect Ollama to the network:

```bash
docker network connect n8n_network ollama
```

---

## 6. Verify the Connection Between Containers

Inspect the network to ensure both containers are connected.

```bash
docker network inspect n8n_network
```

You should see **n8n** and **ollama** listed in the containers section.

---

## 7. Manage Models in Ollama

### a. Access the Ollama Container

```bash
docker exec -it ollama bash
```

### b. List Downloaded Models

Inside the **Ollama** container, run:

```bash
ollama list
```

This will display the models you have downloaded.

### c. Download and Run the **deepseek-coder** Model

If the model is not downloaded, run it to start the download:

```bash
ollama run deepseek-coder
```

### d. Remove the **deepseek-coder** Model

If you need to remove the **deepseek-coder** model to free up space or update it, run the following command inside the Ollama container:

```bash
ollama rm deepseek-coder
```

### e. Download a Specific Version of Ollama Model (e.g., version 3.2)

To get a specific version of an Ollama model, replace `MODEL_NAME` with the desired model name:

```bash
ollama run MODEL_NAME:3.2
```

For example, to download version 3.2 of a model named **ollama**, run:

```bash
ollama run llama3.2:1b
```

```bash
ollama run deepseek-r1:1.5b
```

### f. Exit the Container

```bash
exit
```

### e. Verify the Status of the Containers

To check if **Ollama** is running:

```bash
docker ps -a
```

If **Ollama** is stopped, restart it:

```bash
docker start ollama
```
If you want restart **n8n**  stopped, restart it:

```bash
docker stop n8n
```
```bash
docker start n8n
```

---

## 8. Configure n8n to Connect to Ollama

### a. Change the Base URL in the n8n Ollama Node

When configuring the **Ollama node** in **n8n**, ensure the base URL points to the container name **Ollama** instead of `localhost`.

**Correct URL:**

```
http://ollama:11434
```

### b. Verify the Connection from n8n

To ensure **n8n** can communicate with **Ollama**:

1. Access the n8n container:

```bash
docker exec -it n8n bash
```

2. Run a `curl` command to test the connection:

```bash
curl http://ollama:11434
```

If you receive a response, the connection is working correctly.

---

## 9. Troubleshooting

- **Issue:** The **Ollama** container stops automatically.

  - **Solution:** Check the logs to identify the error:
    ```bash
    docker logs ollama
    ```
  - Ensure you have at least **4 GB of RAM** on the VM.

- **Issue:** n8n cannot connect to Ollama.

  - **Solution:** Ensure both containers are on the `n8n_network` and use the URL `http://ollama:11434`.

---

Done! You now have **Ollama** running locally on your VM and connected with **n8n** to automate your workflows.
