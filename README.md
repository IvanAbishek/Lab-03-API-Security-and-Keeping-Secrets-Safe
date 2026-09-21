# Lab 03: API Security and Keeping Secrets Safe

## Aim
To identify and exploit Broken Object Level Authorization (BOLA / IDOR) vulnerabilities within modern RESTful web APIs and execute automated static credential scanning across source code repositories to prevent secret leakage into version control history.

---

## Architecture Diagram

```mermaid
graph TD
    subgraph Client Tier
        A[Attacker / Security Analyst]
        P[Postman Client]
    end

    subgraph API Gateway & Microservices
        GW[API Gateway / Auth Layer]
        API[Identity Microservice]
        DB[(User & Vehicle Database)]
    end

    subgraph Source Control & DevSecOps
        DEV[Developer Workstation]
        REPO[Local / Remote Git Repository]
        TH[TruffleHog Secret Scanner]
    end

    %% BOLA Attack Flow
    A -->|1. Craft Request with Token| P
    P -->|2. GET /user/dashboard with Victim ID| GW
    GW -->|3. Validates Token, Skips Object Auth| API
    API -->|4. Queries Victim Record| DB
    DB -->|5. Returns Victim Data| API
    API -->|6. Unauthorized Response 200 OK| P

    %% TruffleHog Scan Flow
    DEV -->|Commit & Push with Credentials| REPO
    REPO -->|Run Static Scan| TH
    TH -->|Flag Leaked AWS / Token Secrets| A
```

---

## Tools Required
* **Postman Client:** For API testing, header manipulation, and payload crafting.
* **TruffleHog (CLI):** Deep Git-history secret scanner with live API verification.
* **Git:** Version control client.

---

## Execution Steps

### 1. Tool Installation

#### Install Postman
```bash
# Ubuntu / Debian
sudo snap install postman
```

#### Install TruffleHog CLI
```bash
curl -sSfL [https://raw.githubusercontent.com/trufflesecurity/trufflehog/main/scripts/install.sh](https://raw.githubusercontent.com/trufflesecurity/trufflehog/main/scripts/install.sh) | sudo sh -s -- -b /usr/local/bin
```

---

### 2. Broken Object Level Authorization (BOLA) Exploitation

1. Open **Postman** and configure a `GET` request to:
   ```text
   [http://demo.crapi.apisec.ai/identity/api/v2/user/dashboard](http://demo.crapi.apisec.ai/identity/api/v2/user/dashboard)
   ```
2. In the **Headers** tab, supply the authorization header:
   * **Key:** `Authorization`
   * **Value:** `Bearer <CAPTURED_USER_TOKEN>`
3. Tamper with the user identifier in the request parameter to access another user's profile:
   ```text
   [http://demo.crapi.apisec.ai/identity/api/v2/user/dashboard?user_id=8841](http://demo.crapi.apisec.ai/identity/api/v2/user/dashboard?user_id=8841)
   ```
4. Click **Send** and confirm access to the victim's private profile and vehicle records.

---

### 3. Static Secret Scanning with TruffleHog

1. Open your terminal and navigate to the project repository:
   ```bash
   cd ~/api-secret-lab
   ```
2. Execute TruffleHog against the local Git tree:
   ```bash
   trufflehog git file://. --only-verified
   ```
3. Inspect the terminal summary to identify leaked credentials, file locations, and commit metadata.

---

## Screenshots

### 1. Postman BOLA Authorization Bypass
<img width="837" height="302" alt="image" src="https://github.com/user-attachments/assets/18500eb0-34fc-406e-9368-0bea2442b6e3" />


### 2. TruffleHog Verified Secret Detection
<img width="826" height="240" alt="image" src="https://github.com/user-attachments/assets/780fb919-56f0-485e-998a-1e5300a8d248" />


---

## Result
* Successfully probed and exploited a Broken Object Level Authorization (BOLA) flaw in the target API, demonstrating how missing authorization checks allow unauthorized access to sensitive user data.
* Statically analyzed Git commit logs using TruffleHog, identifying and verifying hardcoded cloud service credentials before deployment.
