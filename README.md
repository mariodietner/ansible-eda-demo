# Event‑Driven Ansible (EDA) — Newbie Demo

A tiny, copy‑paste friendly demo that listens for **webhook** events and runs an Ansible playbook when a condition matches.

## What you'll show
1. Start a webhook listener with **ansible‑rulebook**
2. Send a JSON event with `curl`
3. Watch a playbook run and log the event

---

## Prerequisites
* Python 3.9+ and **Java 17+ (JDK)** installed
* Ansible + ansible‑rulebook + runner + EDA collection

### Install (Linux/macOS)
```bash
# Java 17 (examples)
# Fedora/RHEL/CentOS/Rocky
sudo dnf -y install java-17-openjdk python3-pip
export JAVA_HOME=/usr/lib/jvm/jre-17-openjdk

# Ubuntu/Debian
sudo apt-get -y install openjdk-17-jdk python3-pip
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64

# macOS (Homebrew)
brew install openjdk@17 python@3.11
echo 'export PATH="/usr/local/opt/openjdk@17/bin:$PATH"' >> ~/.bash_profile
source ~/.bash_profile
```
```bash
# Tools
pip3 install --user ansible ansible-rulebook ansible-runner
~/.local/bin/ansible-galaxy collection install ansible.eda
~/.local/bin/ansible-rulebook --version
```

## Run the demo
```bash
cd eda-newbie-demo
# 1) Start the rulebook (leave this running)
~/.local/bin/ansible-rulebook       --inventory inventory.ini       --rulebook rulebook.yml       --print-events -vv
```

In a new terminal, send events:

```bash
# 2) Trigger a "high CPU" alert (will run the playbook)
curl -s -X POST -H 'Content-Type: application/json'       -d @payloads/trigger_high_cpu.json http://127.0.0.1:5000/alerts

# 3) Send a non-matching event (will just print event)
curl -s -X POST -H 'Content-Type: application/json'       -d @payloads/trigger_ok.json http://127.0.0.1:5000/alerts
```

## What to point out while it runs
* **Rulebooks** have **sources → rules → actions**
* `condition: event.payload.alert == "cpu_high"` is the filter
* The **event** appears in your playbook under `ansible_eda.event`
* We save the full JSON to `/tmp/eda-demo/` for visibility

## Files
* `inventory.ini` — local host
* `rulebook.yml` — webhook listener + rules
* `playbooks/respond.yml` — action when rule matches
* `payloads/trigger_high_cpu.json` — fires the rule
* `payloads/trigger_ok.json` — doesn't fire, just prints

## Troubleshooting
* If you see Java errors, ensure **JDK 17** and `JAVA_HOME` are set
* If nothing happens, add `--print-events -vv` to see incoming events
* Ports: change `port:` in `rulebook.yml` if 5000 is busy
* Validate Ansible is on PATH: `which ansible`, `ansible --version`

## Optional: show it with Ansible Automation Platform (EDA Controller)
* Commit this folder to a Git repo
* In EDA Controller: create **Project → Decision Environment → Rulebook Activation** that points at this repo and `rulebook.yml`
* Send the same `curl` requests; watch the activation fire

---

## One‑slide summary (speaker notes)
* *“EDA listens for events and reacts with playbooks. Today: a webhook event triggers a tiny remediation.”*
* Show rulebook anatomy (sources → rules → actions)
* Run it, fire a curl → spotlight the matching rule and `respond.yml` output
* Close with where to plug real events (Alertmanager, Kafka, GitHub webhooks, etc.)
