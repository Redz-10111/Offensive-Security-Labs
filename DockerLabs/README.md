```bash
 /$$$$$$$                      /$$                           /$$                 /$$                
| $$__  $$                    | $$                          | $$                | $$                
| $$  \ $$  /$$$$$$   /$$$$$$$| $$   /$$  /$$$$$$   /$$$$$$ | $$        /$$$$$$ | $$$$$$$   /$$$$$$$
| $$  | $$ /$$__  $$ /$$_____/| $$  /$$/ /$$__  $$ /$$__  $$| $$       |____  $$| $$__  $$ /$$_____/
| $$  | $$| $$  \ $$| $$      | $$$$$$/ | $$$$$$$$| $$  \__/| $$        /$$$$$$$| $$  \ $$|  $$$$$$ 
| $$  | $$| $$  | $$| $$      | $$_  $$ | $$_____/| $$      | $$       /$$__  $$| $$  | $$ \____  $$
| $$$$$$$/|  $$$$$$/|  $$$$$$$| $$ \  $$|  $$$$$$$| $$      | $$$$$$$$|  $$$$$$$| $$$$$$$/ /$$$$$$$/
|_______/  \______/  \_______/|__/  \__/ \_______/|__/      |________/ \_______/|_______/ |_______/ 
```

# 🔐 DockerLabs — Offensive Security Labs

> This section documents practical exploitation scenarios conducted within isolated Docker environments on the DockerLabs platform.


## 🎯 Objective

To document realistic exploitation scenarios following a structured offensive security methodology that includes:

- Service enumeration and analysis
- Vulnerability identification and validation
- Controlled exploitation
- Pivoting and attack chaining
- Privilege validation
- Security impact assessment

Each lab reflects the full attack lifecycle, from initial reconnaissance to validated system compromise.
## 📌 Usage Instructions

## 1️⃣ Lab Information

- **Platform:** [DockerLabs](https://dockerlabs.es)
- **Environment Type:** Docker-based containerized lab  
- **Deployment Model:** Local execution via Docker containers  

All laboratories are executed in controlled containerized environments to ensure reproducibility and safe testing conditions.


## 2️⃣ Download

Machines are downloaded directly from the official [DockerLabs](https://dockerlabs.es) platform.


## 3️⃣ Extraction

If required, extract the provided file:

```bash
unzip <machine_name>.zip
```

## 4️⃣ 🐳 Container Deployment

Each machine is deployed locally as a Docker container using the provided script:
```bash
sudo bash auto_deploy.sh <machine_name>.tar
```

## Example Output

### `[+] Image loaded successfully [+] Container started [+] Assigned IP: 172.17.0.X`

Once deployed, the container becomes accessible through the internal IP address assigned by the Docker network.
