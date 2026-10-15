<div align="center">

# 📡 Ping Sweep

**A lightweight Bash script to identify IP addresses that respond to ICMP ping requests on a /24-style network.**

[![Bash](https://img.shields.io/badge/Shell-Bash-4EAA25?style=for-the-badge&logo=gnubash&logoColor=white)](https://www.gnu.org/software/bash/)
[![Linux](https://img.shields.io/badge/Platform-Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)](https://www.kernel.org/)
[![Kali Linux](https://img.shields.io/badge/Security-Kali%20Linux-557C94?style=for-the-badge&logo=kalilinux&logoColor=white)](https://www.kali.org/)
[![ICMP](https://img.shields.io/badge/Networking-ICMP-0A66C2?style=for-the-badge)](https://en.wikipedia.org/wiki/Internet_Control_Message_Protocol)

[Concept](#-concept) · [Features](#-features) · [Installation](#-installation) · [Usage](#-usage) · [How It Works](#-how-it-works)

</div>

---

## 📌 Overview

**Ping Sweep** is a small network-reconnaissance script written in Bash. It loops through host addresses `1` to `254` for a supplied network prefix, sends one ICMP ping request to each address, and filters the output to print addresses that respond.

The project is designed as a beginner-friendly exercise in Bash scripting, basic networking, loops, pipes, and command-line text processing.

> **Authorization notice:** Run this script only on networks you own or have permission to test.

## 💡 Concept

A **ping sweep** (also called an ICMP sweep) is a basic host-discovery technique. Rather than pinging IP addresses one by one, a script checks a range of addresses and reports those that respond to ICMP echo requests.

For example, supplying `192.168.1` builds addresses such as `192.168.1.1`, `192.168.1.2`, and so on, up to `192.168.1.254`.

A ping response indicates that an address responded to the probe. A lack of response does **not** prove that a device is offline; firewalls, network rules, packet loss, or disabled ICMP responses may hide reachable devices.

## 🎯 Objectives

- Automate a basic host-discovery check across a range of IPv4 addresses.
- Practise Bash `for` loops and command-line arguments.
- Learn how pipes (`|`) connect command output to text-processing utilities.
- Use `grep`, `cut`, and `tr` to filter ping output.
- Understand background execution with `&`.
- Build a simple networking project suitable for a Linux or cybersecurity learning lab.

## ✨ Features

- **Simple input:** accepts a network prefix as the first argument.
- **Host range:** checks last-octet values from `1` through `254`.
- **Single ping per address:** uses `ping -c 1`.
- **Filtered results:** attempts to print only addresses with a matching ping response.
- **Parallel execution:** launches each ping pipeline in the background.
- **No extra framework:** uses standard Bash and common command-line tools.

## 🧰 Tech Stack

<div align="left">

[![Bash](https://img.shields.io/badge/Bash-4EAA25?style=flat-square&logo=gnubash&logoColor=white)](https://www.gnu.org/software/bash/)
[![Linux](https://img.shields.io/badge/Linux-FCC624?style=flat-square&logo=linux&logoColor=black)](https://www.kernel.org/)
[![Kali Linux](https://img.shields.io/badge/Kali%20Linux-557C94?style=flat-square&logo=kalilinux&logoColor=white)](https://www.kali.org/)
[![Git](https://img.shields.io/badge/Git-F05032?style=flat-square&logo=git&logoColor=white)](https://git-scm.com/)

</div>

| Technology / command | Role |
|---|---|
| **Bash** | Runs the loop and coordinates the command pipeline |
| **ping** | Sends one ICMP echo request to each generated IP address |
| **grep** | Filters output lines containing `64 bytes` |
| **cut** | Extracts the fourth space-delimited field |
| **tr** | Removes the colon character from the extracted field |
| **Linux / Kali Linux** | Recommended environment for running the script |
| **Git / GitHub** | Version control and project hosting |

## ⚙️ Installation

### 1. Clone the repository

```bash
git clone https://github.com/gksoni19/ping-sweep.git
cd ping-sweep
```

### 2. Locate the script

Make sure the Bash script is in the repository directory. If the filename is `ping-sweep.sh`, use that name in the commands below. If your script has a different filename, substitute the actual filename.

### 3. Make it executable

```bash
chmod +x ping-sweep.sh
```

### 4. Check the required commands

The script uses Bash, `ping`, `grep`, `cut`, and `tr`. These are available by default on many Linux distributions. If `ping` is missing on Kali or Debian-based Linux, install the relevant package:

```bash
sudo apt update
sudo apt install iputils-ping
```

## 🚀 Usage

### Syntax

```bash
./ping-sweep.sh <first-three-octets>
```

Provide the first three octets of the IPv4 subnet as the first argument. The script appends `.1` through `.254` to that prefix.

### Example

For a network you own or are authorised to test:

```bash
./ping-sweep.sh 192.168.1
```

The script will check addresses from `192.168.1.1` through `192.168.1.254`.

### Save results to a file

```bash
./ping-sweep.sh 192.168.1 > responding-hosts.txt
```

This redirects standard output to `responding-hosts.txt`. Because the script runs its ping pipelines in the background, wait for the background jobs to finish before relying on the file's contents. The current script does not explicitly wait for all jobs.

### Example output

```text
192.168.1.1
192.168.1.12
192.168.1.25
```

These are illustrative examples only. Actual results depend on the network and which devices respond to ping.

## 🔍 How It Works

The script:

```bash
#!/bin/bash

for ip in {1..254}; do
    ping -c 1 "$1.$ip" | grep "64 bytes" | cut -d " " -f 4 | tr -d ":" &
done
```

| Part | Explanation |
|---|---|
| `#!/bin/bash` | Selects Bash as the script interpreter |
| `for ip in {1..254}` | Loops through host values 1 to 254 |
| `$1` | Refers to the first argument supplied when running the script |
| `"$1.$ip"` | Builds each address from the supplied prefix and current host value |
| `ping -c 1` | Sends one ping request |
| `grep "64 bytes"` | Keeps lines containing the text `64 bytes` |
| `cut -d " " -f 4` | Extracts the fourth space-separated field |
| `tr -d ":"` | Removes colons from the extracted text |
| `&` | Runs each pipeline in the background so multiple checks can overlap |

### Workflow

1. Pass a network prefix as a command-line argument.
2. Generate host suffixes from `1` to `254`.
3. Construct an IP address for each suffix.
4. Send one ping request to that address.
5. Filter matching response output and print the extracted address.

## ⚠️ Limitations and Notes

- **Assumes a `/24`-style prefix:** the script always checks suffixes `1` to `254`; it does not calculate arbitrary CIDR ranges.
- **No input validation:** an empty or malformed argument can produce invalid targets.
- **ICMP may be blocked:** a device can be online but not respond to ping.
- **Output parsing is platform-dependent:** ping output format can differ between operating systems and locales, so matching `64 bytes` and extracting field four may not work everywhere.
- **Background jobs are not collected:** `&` starts each pipeline in the background, but the script does not explicitly wait for every job to finish.
- **No rate limiting:** many ping pipelines can run at once, which may generate a burst of traffic on the network.
- **No guarantee of complete discovery:** this is a simple learning tool, not a definitive inventory scanner.

## 🔐 Legal and Ethical Use

This project is intended for learning, network troubleshooting, and authorised host discovery.

- Use it only on networks you own or have explicit permission to test.
- Follow applicable laws and your organisation's policies.
- Avoid running scans against public or third-party networks without permission.
- Remember that network monitoring systems may log ICMP probes.

## 🛣️ Possible Improvements

- Validate the supplied IP prefix before starting.
- Add a helpful usage message when no argument is provided.
- Use more robust parsing that does not depend on the exact `ping` output wording.
- Add a `wait` step so the script can finish after all background jobs complete.
- Limit concurrency or add a short delay between requests.
- Support CIDR notation and custom host ranges.
- Add an option to save results to a timestamped report.

## 🤝 Contributing

Suggestions, bug reports, and improvements are welcome. Open an issue or submit a pull request with a clear description of the proposed change.

## 👤 Author

**gksoni19**

- GitHub repository: https://github.com/gksoni19/ping-sweep

---

<div align="center">

**Built to practise Bash scripting and network fundamentals.**

</div>
