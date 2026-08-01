# TryHackMe: Room 404 — Walkthrough & Solution

## 📌 Overview

- Room Name: Room 404

- Category: Web Security

- Difficulty: Very Easy

- Key Vulnerability: Exposed `.git` Repository (Source Code Disclosure)

## 🔍 Vulnerability Analysis
The Byte Lotus web application running on port `8080` contains an exposed Git repository (`/.git/`) left publicly accessible in the production build.

When developers deploy application source code directly from a version control folder without removing or restricting access to the `.git` directory, unauthorized users can enumerate, dump, and reconstruct the full repository structure. This leads to complete source code exposure and leakages of internal configuration files, staging credentials, or hardcoded sensitive data.

## 🚀 Step-by-Step Exploitation

### Step 1: Directory Enumeration
We begin by running a web directory brute-force scan against the target application (`http://10.112.174.188:8080`) using `gobuster` to identify hidden endpoints and files.

```bash
gobuster dir -u http://10.112.174.188:8080 -w /usr/share/wordlists/dirb/common.txt -x php,txt,html,bak,zip
```

The scan discovers an exposed Git repository file located at `/.git/HEAD` returning a HTTP 200 OK status code.

### Step 2: Manual Verification of the Exposed Git Directory
To manually verify the existence of the exposed `.git` directory, we navigate to `http://10.112.174.188:8080/.git/HEAD` in the browser.

Opening the downloaded HEAD file reveals its contents: `refs/heads/main`

Navigating to `http://10.112.174.188:8080/.git/refs/heads/main` allows us to retrieve the commit hash associated with the primary branch `0f13550b4cb13e9f30c61d5b342c532d21e45bda`.

### Step 3: Installing git-dumper Tool
To dump the entire exposed Git structure automatically and rebuild the local repository source files, we install git-dumper via Python package manager (pip).

Bash

```pip install git-dumper```

### Step 4: Dumping the Source Code Repository
With `git-dumper` ready, we execute it against the target host to recursively fetch all Git objects and restore the working directory into a local folder named room404.

Bash

```git-dumper http://10.112.174.188:8080/.git room404```

### Step 5: Extracting the Flag from Source Code
After downloading the repository, we navigate into the reconstructed room404 directory and list its contents:

Bash
```
cd room404
ls
```
The repository contains `README.md, app.js, and index.html`. Inspecting `README.md` reveals internal developer notes and the target staging flag left prior to launch:

Bash

```cat README.md```

## 🚩 Result & Flag

Exposing the internal staging repository revealed the hardcoded staging flag embedded inside the README.md file.

`THM{************}`

