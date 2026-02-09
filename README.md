# 🚗🔐 AutoSec - Connected Vehicle Intrusion Detection System (IDS)

AutoSec is a final-year project exploring intrusion detection for connected vehicle networks. The system passively monitors CAN bus traffic in a simulated in-vehicle network, detects suspicious activity using signature-based rules and anomaly detection, and streams real-time alerts to a web dashboard for monitoring and analysis.

Focus: Automotive cybersecurity · CAN bus · Intrusion Detection Systems (IDS) · Real-time alerting · Web dashboard

⸻

🔎 Overview

Modern vehicles rely heavily on in-vehicle networks (e.g., CAN bus), which were not designed with security in mind. AutoSec demonstrates how an IDS can be deployed alongside a vehicle network (simulated using ICSim and virtual CAN) to:
	•	Observe live CAN traffic (vcan0)
	•	Detect common attack patterns (replay, injection, flooding)
	•	Generate alerts with low latency
	•	Visualise events on a web-based dashboard

This project is intended as a research and educational prototype, aligned with concepts from UNECE R155 and ISO/SAE 21434.

⸻

🧩 System Architecture

Pipeline:

ICSim / CAN Simulator
→ CAN Sniffer (vcan0 listener)
→ IDS Engine (rules + anomaly detection)
→ Alert API (HTTP/WebSocket)
→ Web Dashboard (Netlify-hosted)

Key Components:
	•	CAN Sniffer: Passive listener for vcan0 traffic
	•	Rule Engine: Signature-based detection
	•	Anomaly Detection: Baseline deviation monitoring
	•	Alert Service: Streams events to UI
	•	Dashboard: Live monitoring, logs, and metrics

⸻

⚙️ Tech Stack
	•	Simulation: ICSim, virtual CAN (vcan0)
	•	IDS Engine: Python (Linux)
	•	Web Dashboard: HTML, CSS, JavaScript
	•	Deployment: Netlify (frontend)
	•	Tooling: Git, VS Code, Linux

⸻

🧪 Scenarios & Features

Attack Scenarios (simulated):
	•	Replay attacks
	•	CAN flooding / DoS
	•	Spoofed ECU messages
	•	Diagnostic misuse (UDS)

Features:
	•	Real-time CAN traffic monitoring
	•	Signature-based detection rules
	•	Alert streaming to web UI
	•	Logging & replay of events
	•	Metrics: detection latency, false positives (demo-scale)

⸻

🚀 Getting Started (High-level)

Detailed setup docs will be added as the project matures.

	1.	Set up Linux environment with virtual CAN (vcan0) and ICSim
	2.	Run CAN simulator and traffic generator
	3.	Start IDS service (sniffer + detection pipeline)
	4.	Launch web dashboard locally or view hosted demo
	5.	Trigger simulated attacks and observe alerts

⸻

🌐 Live Demo
	•	Web Dashboard: https://autosec.netlify.app
	•	Repository: https://github.com/LuluLogics/AutoSec

⸻

📸 Screenshots & Media

Screenshots of:

	•	Web dashboard UI
	<img width="3420" height="1906" alt="image" src="https://github.com/user-attachments/assets/2eb45ff8-fb05-4542-a798-b3feff4e0bfd" />

	•	Architecture diagram
	•	IDS running in terminal
	•	CAN traffic / alerts

⸻

🎓 Academic Context

This project is developed as part of a BSc (Hons) in Software Development final-year project at Munster Technological University (MTU).
It aims to bridge automotive security research with practical IDS prototyping.

⸻

🛣️ Roadmap
	•	Expand detection ruleset
	•	Add anomaly detection models
	•	Persist alerts to database
	•	Add authentication for API
	•	Improve metrics & evaluation
	•	Document full reproducible setup

⸻

⚠️ Disclaimer

AutoSec is a research prototype for educational purposes.
It is not production-ready and should not be used as a real vehicle security system.

⸻

👤 Author

Luté Abrahams
	•	GitHub: https://github.com/LuluLogics
	•	Portfolio: https://lululogics.netlify.app
	•	LinkedIn: https://www.linkedin.com/in/l-abrahams-439567208/
