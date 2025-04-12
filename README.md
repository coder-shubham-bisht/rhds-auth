## Authentication Flow from RHDS to RHEL Client During Login

### Overview
When a user logs in to a RHEL client system that is connected to a Red Hat Directory Server (RHDS) via LDAP, several components work together to authenticate the user and initialize their session. Below is a step-by-step explanation of the components involved and their sequence.

---

### Components Involved
- **PAM (Pluggable Authentication Modules)**
- **SSSD (System Security Services Daemon)**
- **NSS (Name Service Switch)**
- **LDAP Protocol**
- **RHDS (Red Hat Directory Server)**
- **TLS/SSL** (for secure LDAP communication)

---

### Step-by-Step Login Sequence

#### 1. User Enters Credentials
- The user inputs a username and password at the login screen (e.g., graphical login manager or terminal).
- These credentials are passed to the authentication stack.

#### 2. PAM Executes Authentication Modules
- PAM configuration (e.g., `/etc/pam.d/system-auth`) defines how authentication should be handled.
- PAM uses modules like:
  - `pam_sss.so` for SSSD-based authentication
  - `pam_unix.so` for local `/etc/shadow` authentication

#### 3. SSSD Handles Authentication
- If `pam_sss.so` is used, PAM delegates the authentication to SSSD.
- SSSD checks if credentials are cached:
  - If cached: SSSD authenticates offline.
  - If not cached: SSSD connects to RHDS via LDAP.
- SSSD performs a **bind operation** using the user's DN and password to validate credentials.

#### 4. RHDS Verifies Credentials
- RHDS receives the LDAP bind request.
- If the username and password match an entry in RHDS, the bind succeeds.
- If not, authentication fails.

#### 5. SSSD Returns the Result to PAM
- SSSD informs `pam_sss.so` of the result.
- PAM continues based on success/failure of authentication.

#### 6. NSS Resolves User Information
- After successful authentication, the system uses NSS to retrieve user details:
  - UID, GID
  - Home directory
  - Shell
- NSS configuration in `/etc/nsswitch.conf` directs the system to query:
  - Local files (e.g., `/etc/passwd`)
  - SSSD (which queries RHDS)

#### 7. PAM Session Modules Run
- PAM runs session modules to prepare the environment:
  - `pam_mkhomedir.so` to create a home directory if needed
  - Logging, auditing, Kerberos ticket management, etc.

#### 8. User Session Starts
- With authentication and user setup complete, the system starts the user session:
  - Terminal shell or desktop environment is presented

---

### Summary Table: Component Sequence

| Step | Component | Role |
|------|-----------|------|
| 1 | Display Manager or `login` | Captures credentials |
| 2 | PAM | Handles auth policy, invokes modules |
| 3 | `pam_sss.so` | Delegates to SSSD |
| 4 | SSSD | Contacts RHDS via LDAP, performs bind |
| 5 | RHDS (LDAP) | Validates password (via bind) |
| 6 | NSS + SSSD | Fetch user info (UID, home, etc.) |
| 7 | PAM Session Modules | Create home dir, initialize session |
| 8 | User Shell/Desktop | Finalize login process |

---

### Optional Enhancements
- **TLS/SSL**: Ensure secure communication between client and RHDS.
- **Kerberos Integration**: If RHDS is integrated with Kerberos, SSSD may also obtain tickets during login.

---

### Configuration Files of Interest
- `/etc/sssd/sssd.conf`
- `/etc/nsswitch.conf`
- `/etc/pam.d/system-auth`
- `/etc/pam.d/password-auth`
- `/etc/openldap/ldap.conf` (if using LDAP directly)

---

This document outlines the core components and sequence involved in authenticating a user from RHDS to a RHEL client system.

