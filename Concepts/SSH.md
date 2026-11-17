___

# Level 1 - Foundations
**SSH = Secure Shell.**
SSH is a protocol used to securely access and control another computer over a network.

Think of it as:
```
Your Machine
     |
     | Encrypted SSH connection
     v
Remote Linux Server
```

> Instead of physically sitting at the remote server, you get a terminal over the network.

Default SSH port : ***TCP 22***

### Core Concepts
| Section            | Explanation                                                                                                                 |
| ------------------ | --------------------------------------------------------------------------------------------------------------------------- |
| Simple Meaning     | SSH gives you secure remote command-line access to another computer.                                                        |
| Purpose            | Remote administration, file transfer, tunneling, automation, Git access, server management.                                 |
| Mental Model       | An encrypted tunnel connecting your terminal to a remote machine.                                                           |
| Technical View     | A TCP-based client/server protocol that establishes an encrypted session and authenticates the server and usually the user. |
| Security Relevance | SSH can provide direct system access, so authentication, keys, configuration, and exposed services matter enormously.       |
### The main components are:
```
SSH Client
SSH Server
SSH Protocol
Authentication
Encryption
Keys
```

| Component | Description | Common Examples | Key Detail |
|---|---|---|---|
| **SSH Client** | The program initiating the SSH connection | OpenSSH `ssh`, PuTTY, Windows OpenSSH, SSH libraries used by automation software | `ssh` = client |
| **SSH Server** | The remote service accepting SSH connections | OpenSSH Server | `sshd` = server daemon |
### Normal Working
A simplified SSH login looks like this:
```
Client                          SSH Server
  |                                 |
  | ------ TCP Connection --------> |
  |                                 |
  | <---- SSH Identification ------ |
  |                                 |
  | ---- Algorithm Negotiation ---> |
  |                                 |
  | <--- Server Authentication ---- |
  |                                 |
  | ==== Encrypted Channel ======== |
  |                                 |
  | ---- User Authentication -----> |
  |                                 |
  | <------ Access Granted -------- |
  |                                 |
  | ===== Remote Shell ============ |
```

| Stage | What Happens | Important Details | Security Mindset |
|---|---|---|---|
| **1. Network Connection** | The SSH client establishes a TCP connection to the SSH server | Usually `22/tcp`, but SSH can run on any TCP port such as `2222`, `22022`, or `443` | **Port 22 closed ≠ SSH does not exist.** Identify the service rather than assuming from the port |
| **2. SSH Identification** | Client and server exchange information about their SSH implementations | Example: `SSH-2.0-OpenSSH_x.x`; may reveal protocol version, implementation, version, and sometimes OS packaging information | A banner is **evidence, not proof** of a vulnerability. Backported patches can make version matching unreliable |
| **3. Cryptographic Negotiation** | Client and server agree on supported cryptographic algorithms | Covers **key exchange**, **server authentication**, **encryption**, and **integrity protection** | Only algorithms supported by both sides can be selected |
| **4. Server Authentication** | The client verifies the identity of the SSH server using its **host key** | The client may warn: `The authenticity of host ... can't be established.` The host key can be stored for future connections | A changed host key can indicate reinstallation, key rotation, configuration changes, a different backend, DNS/IP changes, or potentially a **Man-in-the-Middle attack** |
| **5. User Authentication** | The server verifies whether the connecting user is allowed to log in | Common methods: **password**, **public key**, **keyboard-interactive**, and **MFA** | **Server authentication ≠ User authentication**. One verifies the server; the other verifies the user |
| **6. Encrypted Channel / Remote Shell** | After negotiation and authentication, SSH provides an encrypted session and the user can interact with the remote system | Commands, terminal input/output, and other SSH traffic travel through the protected channel | The connection is now an authenticated, encrypted SSH session |
#### Important Terminology
##### SSH
The overall protocol.
```
Secure Shell
```

##### OpenSSH
A widely deployed SSH implementation.
SSH is the protocol.
OpenSSH is software implementing that protocol.

Similar idea:
```
HTTP        = protocol
Apache      = implementation

SSH         = protocol
OpenSSH     = implementation
```

##### sshd
The SSH server daemon.
```
sshd
```
usually waits for incoming SSH connections.

## Security Perspective
From an OSCP perspective, seeing:
```
22/tcp open ssh
```
should start a reasoning tree.

```
SSH discovered
     |
     ├── What implementation?
     |
     ├── What version?
     |
     ├── What authentication methods?
     |
     ├── What usernames might exist?
     |
     ├── Are credentials available elsewhere?
     |
     ├── Are private keys exposed?
     |
     ├── Is password authentication enabled?
     |
     └── Is configuration weak?
```

Do **not** think:
```
SSH open → exploit SSH
```

Think:
```
SSH open
   ↓
Remote authentication interface exists
   ↓
Gather information
   ↓
Look for valid authentication paths
```

### Practical Relevance
Imagine an OSCP lab machine exposing:
```
22/tcp   SSH
80/tcp   HTTP
```

A weak approach:
```
Try random SSH exploits.
```

A stronger approach:
```
Enumerate SSH
      +
Enumerate web server
      ↓
Discover username from website
      ↓
Discover credential from web configuration
      ↓
Test whether credential belongs to SSH account
```
