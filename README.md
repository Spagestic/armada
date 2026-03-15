<div align="center">

<a id="logo" href="#logo"><img src="assets/logo.png" alt="Armada Logo" width="120"/></a>

# Armada

### Scale Your Bots & Scrapers. Effortlessly.

<img src="https://img.shields.io/badge/version-0.9.0-383b42?style=flat&logoColor=8b949e" alt="Version 0.9.0"/>&nbsp;![License: AGPL v3](https://img.shields.io/badge/License-AGPL%20v3-blue.svg)&nbsp;<a href="https://armada.services/docs"><img src="https://img.shields.io/badge/See_The_Full_Documentation-383b42?style=flat&logo=readthedocs&logoColor=8b949e" alt="Link to Full Documentation"/></a>

**Write once, deploy in seconds. Let the orchestrator handle the rest.** 
<br>
<sub>*⇨ See the full documentation [here](https://armada.services/docs)*</sub>

</div>

<div align="center">
<a href="#diagram-deploy"><img src="assets/deployment-flow.svg" alt="Kubernetes Deployment Flow" width="900"/></a>
</figure>
</div>

## 📖 What is Armada?

Armada is a **Kubernetes-native framework for bot and scraper orchestration**. 

When building web scrapers or automated bots, writing the script is only 10% of the work. The other 90% is infrastructure: distributing tasks across hundreds of workers, rotating proxies, managing browser fingerprints, handling job queues, and avoiding bot detection. 

Armada handles the heavy lifting of infrastructure so you can focus purely on your automation logic. You provide a central configuration and your Python script, and Armada provisions the resources, scales the workers, and executes the jobs seamlessly across a Kubernetes cluster.

### ✨ Key Features
* **Built-in Scale-Out Distribution:** Automatically distributes jobs via Redis and RabbitMQ to as many pods as you need.
* **Batteries-Included Services:** Comes out of the box with Proxy rotation, Browser Fingerprint curation, and a live Monitoring Dashboard.
* **Fantomas Integration:** Built-in advanced browser automation library with human emulation (curved mouse trajectories, randomized typing) to bypass anti-bot systems.

---

## 🚀 Demo

https://github.com/user-attachments/assets/ca76f5cc-3931-4e15-8875-41638bc72674

---

## 🛠️ First Try — Reproduce the Demo Locally

*For a proper Quick-start on a real K8s cluster, see the [Quick Start - Installation Guide](https://armada.services/docs/getting-started/quickstart-kubernetes/).*

In this quickstart, you will launch a local Kubernetes environment that simulates a distributed automation task. Armada will provision 3 agents to execute 10 automated messages in parallel.

**Requirements**: Linux, Python, Azure SQL Database instance, Docker, Kubernetes on Minikube (local) with Kubectl and Helm.

### 1. Expose the testing app locally with Docker

First, spin up the local target application that our bots will interact with:

```bash
cd first-try/first-try-website
docker build -t twittor .
docker run -p 5010:5010 twittor
cd ../..
```
*The App becomes accessible at http://localhost:5010 (or http://host.minikube.internal:5010 in Minikube)*

### 2. Complete `.env` with mandatory values

```dotenv
SQL_SERVER_USER=your_user
SQL_SERVER_PASSWORD=your_password
SQL_SERVER_DB=your_database
SQL_SERVER_NAME=your_server.database.windows.net
DOCKER_HUB_USERNAME=armadasvc
```
*See [Environment Variables](https://armada.services/docs/reference/architecture/environment-variables/) for the complete reference.*

### 3. Bootstrap resources

This provisions the necessary databases, secrets, and core Kubernetes resources for Armada:

```bash
cd bootstrap
pip install -r requirements.txt
python bootstrap_database.py
python bootstrap_secrets.py
python bootstrap_cluster_resources.py
cd ..
```
*Select option 3 when prompted. See [Bootstrap Scripts](https://armada.services/docs/reference/architecture/bootstrap-scripts/) for details.*

### 4. Create your first project using first-try mode

```bash
bash create-project.sh
```
**Choose option 2 (First Try project)**, and select a directory to store the project folder. 
*(Note: Python, JSON, and CSV files are ready to run automatically for this config—10 messages across 3 agents).*

For production project setups, see [Setting Up a Project](https://armada.services/docs/setting-up-project/).

### 5. Launch and monitor

In the **Launch** tab, open the Armada Dashboard:

```bash
kubectl port-forward svc/armada-frontend 8080:8080
```

1. Drag and drop your `first-try-project` folder into the dashboard and click **Launch**.
2. Switch to the **Monitor** tab to watch your run in real-time.

*Note: On the very first run without a pre-existing cache, there will be a cold start. The cluster spins up heavy resources (such as the browser), so it is normal to see `No Jobs` for a short period.*

---

## ⚙️ Under the Hood

### 1. A Batteries-Included Service Ecosystem

Armada ships with a full suite of dedicated microservices that agents can leverage out of the box. A Proxy Service selects, rotates, and curates proxies, while a Fingerprint Service fetches and repackages browser fingerprint tokens — both backed by a shared SQL Server. The Orchestrator sits at the center, coordinating everything, and a Monitoring Service feeds real-time job and event data down to a live Dashboard.

<p id="diagram-arch" align="center">
  <a href="#diagram-arch"><img src="assets/architecture.svg" alt="Full System Architecture" width="900"/></a>
</p>

### 2. A Simple Yet Powerful Input Pipeline
The Armada Orchestrator ingests three user-provided inputs: arbitrary Python code for automation logic, a central JSON configuration for infrastructure and agent/job defaults, and optional CSV files for fine-grained per-agent or per-job overrides. Then, it merges them together into fully resolved, executable tasks that are distributed to Kubernetes pods.

<p id="diagram-config" align="center">
  <a href="#diagram-config"><img src="assets/config-pipeline.svg" alt="Configuration Pipeline" width="850"/></a>
</p>

### 3. Built-In Scale-Out Distribution

Scaling from one to hundreds of workers requires zero changes to your code. The Orchestrator handles the heavy lifting: it first seeds each pod with its own tailored configuration via Redis, then floods a RabbitMQ queue with jobs that pods consume on demand. Faster workers naturally pick up more tasks, ensuring optimal throughput.

<p id="diagram-queue" align="center">
  <a href="#diagram-queue"><img src="assets/distribution.svg" alt="Redis & RabbitMQ Flow" width="920"/></a>
</p>

---

## 🎭 Supported Drivers & Introducing Fantomas

<p align="center">
  <img src="assets/supported-drivers.png" alt="Supported Drivers: Playwright, Selenium, nodriver, Fantomas" width="700"/>
</p>

### Fantomas 
**Fantomas is Armada's in-house browser automation library, built on top of nodriver and purpose-built to run inside agent pods.**

- **Human emulation out of the box:** Every click follows a physics-based curved trajectory (WindMouse), and every keystroke is typed with randomized delays — no teleporting cursors or instant text injection.
- **Two interaction strategies:** Native mode operates through Chrome's CDP protocol with emulation layers on top. XDO mode goes deeper, firing events at the OS level via xdotool, making them virtually indistinguishable from real user input.
- **Seamless Armada integration:** Designed around Armada's two-tier lifecycle: a browser instance is launched once at the agent level and reused across all jobs, avoiding the cost of spinning up Chrome for every single task.
- **Full nodriver compatibility:** Every native nodriver method (`get`, `query_selector`, `tabs`, ...) remains directly available. Fantomas extends the API without replacing it. 

*See [Fantomas Documentation](https://armada.services/docs/fantomas/) and [API Reference](https://armada.services/docs/fantomas/fantomas-reference/).*

<p id="fantomasdoc" align="center">
  <a href="#fantomasdoc"><img src="assets/fantomas.svg" alt="Fantomas Lib" width="920"/></a>
</p>

---

## 📚 Documentation

For detailed guides, API reference, and advanced configuration, visit the **[Full Documentation](https://armada.services/docs)**.

## 🚀 Initial Public Release 

We are starting with a clean Git history for this first public release (0.9.0), aligning the project with a fresh SemVer strategy and avoiding exposure of legacy commits or releases that no longer reflect the current state of the codebase.

## 🤝 Contributing

Please read [CONTRIBUTING.md](CONTRIBUTING.md) before submitting a pull request.

## 📜 Code of Conduct

This project follows the guidelines defined in [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md).

## ⚖️ License

See [LICENSE](LICENSE) for details.
