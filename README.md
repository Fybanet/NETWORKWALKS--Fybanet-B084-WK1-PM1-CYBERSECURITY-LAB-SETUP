# 🔐 Cybersecurity Lab Setup Using VirtualBox and Kali Linux

**Project:** NetworkWalks — B084 | WK1-PM1  
**Focus:** Virtual Machine Setup, NAT Network Configuration, and DNS Troubleshooting

## 📌 Project Overview

This project documents the setup of a virtual cybersecurity lab using Oracle VirtualBox and Kali Linux on an Ubuntu host operating system.

The objective was to establish a functional virtual environment for cybersecurity learning, ethical hacking, and authorized penetration-testing practice.

During the network configuration stage, Kali Linux lost its ability to access the internet through the browser after the VirtualBox NAT Network settings were configured. Initial connectivity tests showed that the virtual machine could communicate with an external IP address, but domain names could not be resolved.

The issue was diagnosed as a DNS resolution problem and resolved by configuring DNS servers through NetworkManager using `nmcli`.

This README documents the environment, configuration, troubleshooting process, commands used, and the final results.

## 🎯 Project Objectives

- Install and configure VirtualBox on an Ubuntu host.
- Download and install Kali Linux as a virtual machine.
- Configure a VirtualBox NAT Network.
- Configure IPv4 networking for Kali Linux.
- Test IP connectivity and DNS resolution.
- Troubleshoot internet connectivity problems.
- Restore browser access by correcting DNS configuration.
- Document the process using screenshots and terminal commands.

## 🛡️ Purpose of the Lab

The lab can be used for activities such as:

- Network reconnaissance
- Port scanning
- Vulnerability assessment
- Packet analysis
- Web security testing
- Exploitation practice
- Security-tool experimentation

⚠️ Important: This lab must only be used on systems that you own or have explicit permission to test. Do not use the lab or its tools against unauthorized systems.

## 🖥️ Lab Environment

| Component | Configuration |
|---|---|
| Host Operating System | Ubuntu 26.04.1 LTS |
| Virtualization Platform | Oracle VirtualBox 7.2.6_Ubuntu |
| Guest Operating System | Kali-Linux-2026.2 |
| Virtual Network | VirtualBox NAT Network |
| IPv4 Address | `10.0.0.2/24` |
| Subnet Mask | `255.255.255.0` |
| Default Gateway | `10.0.0.1` |
| Primary DNS | `8.8.8.8` |
| Secondary DNS | `1.1.1.1` |
| Network Configuration Tool | NetworkManager (`nmcli`) |
| Browser Used for Testing | Brave-browser |

> **Note:** The IP address, gateway, and DNS values above reflect the configuration shown or described during this lab. Confirm that the VirtualBox NAT Network uses the same subnet before applying these settings to another machine.

## 🏗️ Lab Architecture

```text
              Ubuntu Host OS
                    |
              Oracle VirtualBox
                    |
              Virtual Machine
                    |
                Kali Linux
                    |
             Virtual Network
                    |
              VirtualBox NAT
                    |
              Internet Access
                    |
        External IP and DNS Servers
```

The NAT Network provides network address translation for connected virtual machines and can allow them to communicate with each other when configured on the same network.

## 🛠️ Lab Setup Procedure

### Step 1: Install the Host Virtualization Environment

VirtualBox was installed on the Ubuntu host operating system to provide the environment for running Kali Linux as a virtual machine.

Kali Linux was downloaded and installed as the guest operating system inside VirtualBox.

Official resources:

- [Oracle VirtualBox Downloads](https://www.virtualbox.org/wiki/Downloads)
- [Kali Linux Downloads](https://www.kali.org/get-kali/)

### Step 2: Configure the VirtualBox NAT Network

The VirtualBox networking configuration was changed to use a NAT Network.

The intended network configuration was:

| Setting | Value |
|---|---|
| Network Type | NAT Network |
| IPv4 Network | `10.0.0.0/24` |
| Kali Linux IP | `10.0.0.2` |
| Default Gateway | `10.0.0.1` |

The NAT Network configuration was intended to provide the virtual machine with external network connectivity while allowing additional lab machines to be connected to the same virtual network.

**Important:** The guest IP address and gateway must match the actual NAT Network configuration in VirtualBox.

### Step 3: Configure Kali Linux IPv4 Settings

The IPv4 settings for `Wired connection 1` were configured manually using NetworkManager.

The configuration shown in the network settings was:

- **Method:** Manual
- **Address:** `10.0.0.2`
- **Netmask:** `/24`
- **Gateway:** `10.0.0.1`
- **DNS servers:** `8.8.8.8, 1.1.1.1`

Screenshot:

![Kali Linux IPv4 Network Configuration](https://github.com/Fybanet/NETWORKWALKS--Fybanet-B084-WK1-PM1-CYBERSECURITY-LAB-SETUP/blob/b939094f50ca65305419c972f6492734abe96a8a/IPv4%20Network%20Configuration.png)

This configuration assigns the virtual machine a consistent IPv4 address and specifies the gateway and DNS resolvers it should use.

### Step 4: Troubleshoot the Initial Connectivity Issue

After configuring the NAT Network, Kali Linux could no longer access websites through the browser.

The browser displayed a DNS-related error indicating that the website's server IP address could not be found.

To investigate, I tested connectivity to Google's public DNS IP address.

```bash
ping 8.8.8.8
```

The test returned successful replies:

```text
3 packets transmitted, 3 received, 0% packet loss
```

This indicated that IP connectivity to `8.8.8.8` was working at the time of the test.

Next, I tested domain name resolution:

```bash
ping google.com
```

The command returned:

```text
ping: google.com: Temporary failure in name resolution
```

This showed that Kali Linux could reach an external IP address but could not resolve the hostname `google.com`.

The investigation therefore focused on DNS configuration rather than assuming that all internet connectivity was unavailable.

### Step 5: Adjust the NetworkManager Connection

I first applied the following NetworkManager setting:

```bash
sudo nmcli connection modify "Wired connection 1" ipv4.dad-timeout 0
```

I then restarted the connection:

```bash
sudo nmcli connection down "Wired connection 1"
sudo nmcli connection up "Wired connection 1"
```

The `ipv4.dad-timeout` setting changes the duplicate-address-detection timeout associated with the connection. It is not a DNS setting, so it did not by itself resolve the hostname resolution failure.

The successful ping to `8.8.8.8` helped isolate the remaining problem to DNS resolution.

### Step 6: Configure DNS Servers

To resolve the DNS problem, I configured Google Public DNS and Cloudflare DNS on the active NetworkManager connection.

**Set the DNS servers:**

```bash
sudo nmcli connection modify "Wired connection 1" ipv4.dns "8.8.8.8 1.1.1.1"
```

**Prevent automatically supplied IPv4 DNS servers from overriding the manual configuration:**

```bash
sudo nmcli connection modify "Wired connection 1" ipv4.ignore-auto-dns yes
```

**Restart the connection to apply the changes:**

```bash
sudo nmcli connection down "Wired connection 1"
sudo nmcli connection up "Wired connection 1"
```

These commands configure the connection to use the specified DNS servers instead of DNS servers automatically supplied through IPv4 network configuration.

### Step 7: Verify DNS Resolution

After restarting the connection, I checked the resolver configuration:

```bash
cat /etc/resolv.conf
```

I then tested hostname resolution:

```bash
ping google.com
```

The DNS configuration change resolved the issue, restoring the ability to resolve domain names and access websites through the browser.

> The exact contents of `/etc/resolv.conf` can vary because Kali Linux uses NetworkManager and may use a dynamically managed resolver configuration. Confirm the active DNS settings rather than relying on a particular file format.

## 🔎 Network Verification Commands

The following commands are useful for checking the guest network configuration and diagnosing similar issues.

| Purpose | Command |
|---|---|
| View IP addresses | `ip addr` |
| View routing table | `ip route` |
| View NetworkManager connections | `nmcli connection show` |
| View active network devices | `nmcli device status` |
| View connection DNS settings | `nmcli connection show "Wired connection 1"` |
| Test external IP connectivity | `ping -c 4 8.8.8.8` |
| Test DNS resolution | `getent hosts google.com` |
| Test domain connectivity | `ping -c 4 google.com` |
| Inspect resolver configuration | `cat /etc/resolv.conf` |

### Interpreting the Results

| Test | Result | Interpretation |
|---|---|---|
| `ping 8.8.8.8` | Successful replies | IP connectivity to that destination is working |
| `ping google.com` before the fix | Temporary failure in name resolution | DNS lookup failed |
| DNS configuration updated | `8.8.8.8` and `1.1.1.1` configured | Manual DNS resolvers specified |
| Domain resolution after the fix | Successful | DNS resolution restored |

A successful ping to an IP address does not guarantee that every internet service is working. Similarly, a successful DNS lookup does not by itself guarantee browser connectivity.

## 🐞 Problems Encountered and Solutions

### Problem 1: Loss of Browser Connectivity After NAT Network Configuration

**Symptom:** Websites could not be opened after changing the virtual network configuration.

**Investigation:** Tested connectivity using `ping 8.8.8.8`.

**Finding:** The ping succeeded with zero packet loss during the test.

**Resolution:** Continued troubleshooting DNS resolution and configured the DNS servers manually through NetworkManager.

### Problem 2: Failure to Resolve Domain Names

**Symptom:** The command `ping google.com` returned a temporary failure in name resolution.

**Cause:** DNS resolution was not functioning correctly for the active connection.

**Solution:** Configured Google Public DNS (`8.8.8.8`) and Cloudflare DNS (`1.1.1.1`), disabled automatically supplied IPv4 DNS servers for the connection, and restarted the connection.

**Outcome:** Domain name resolution was restored, resolving the issue that prevented normal browser access.

## 📸 Screenshots

The following screenshots document the troubleshooting process.

### 1. Initial DNS Resolution Failure

The terminal shows successful ping replies from `8.8.8.8`, followed by a name-resolution error when attempting to ping `google.com`.

![Initial connectivity test and DNS failure](https://github.com/Fybanet/NETWORKWALKS--Fybanet-B084-WK1-PM1-CYBERSECURITY-LAB-SETUP/blob/b939094f50ca65305419c972f6492734abe96a8a/Initial%20DNS%20Resolution%20Failure.png)

### 2. IPv4 Network Configuration

The NetworkManager window shows the manually configured IPv4 address, gateway, and DNS servers.

![Manual IPv4 configuration in Kali Linux](https://github.com/Fybanet/NETWORKWALKS--Fybanet-B084-WK1-PM1-CYBERSECURITY-LAB-SETUP/blob/b939094f50ca65305419c972f6492734abe96a8a/IPv4%20Network%20Configuration.png)



## 💡 Key Lessons Learned

Through this lab, I learned:

1. **Virtual machine networking:** How VirtualBox NAT Network settings affect connectivity from a Kali Linux guest.
2. **IPv4 configuration:** How IP addresses, subnet masks, and gateways work together.
3. **Network troubleshooting:** Why it is useful to test IP connectivity separately from domain name resolution.
4. **DNS configuration:** How to configure DNS servers using NetworkManager's `nmcli` commands.
5. **Connection management:** How to restart a NetworkManager connection to apply configuration changes.
6. **Technical documentation:** How to record terminal output, configuration changes, errors, and solutions in a reproducible format.

The most important lesson was that a VM can have working IP connectivity while DNS resolution is failing. Testing these separately makes it easier to identify the source of a networking problem.

## 🔐 Security and Ethical Use

This lab is intended for educational purposes, cybersecurity learning, and authorized testing.

- Perform security testing only on systems you own or have explicit permission to test.
- Keep vulnerable target systems within an appropriately isolated lab network.
- Verify network adapter settings before introducing additional virtual machines.
- Create VirtualBox snapshots at important configuration milestones so that the environment can be restored when needed.

## 📚 Tools and Resources

- [Ubuntu](https://ubuntu.com/)
- [Oracle VirtualBox](https://www.virtualbox.org/)
- [Kali Linux](https://www.kali.org/)
- [NetworkManager Documentation](https://networkmanager.dev/docs/)
- [Google Public DNS](https://developers.google.com/speed/public-dns)
- [Cloudflare DNS](https://one.one.one.one/)

## 📌 Project Information

- **Program:** NetworkWalks
- **Batch:** B084
- **Week:** 01
- **Project Code:** WK1-PM1
- **Project:** Cybersecurity Lab Setup
- **Host OS:** Ubuntu Linux
- **Guest OS:** Kali Linux
- **Repository:** [NETWORKWALKS--Fybanet-B084-WK1-PM1-CYBERSECURITY-LAB-SETUP](https://github.com/Fybanet/NETWORKWALKS--Fybanet-B084-WK1-PM1-CYBERSECURITY-LAB-SETUP)

---

*This documentation records my lab setup, the DNS resolution issue encountered during network configuration, the commands used to troubleshoot it, and the steps taken to restore connectivity.*

