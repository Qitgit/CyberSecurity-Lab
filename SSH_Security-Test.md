# SSH Security test

### 1. Overview
I conducted an experiment to see how 'Linux" handles security events by monitoring the system logs in real-time.
I compared what happens during a login failrue versus a succesful login.

### 2. Environment
- Server : Ubuntu (running on UTM)
- Client : Macbook Terminal via SSH)
- Tool ('tail -f /var/log/auth.log'


---

### 3. Case A : Intentional Login failure
I intentionally entered wrong password three time to trigger the security mechanism.

**observed Log Results**
* 'Failed password for q from 192.168.64.1' : The system explicity recorded each wrong attempt.
* 'PAM 2 more aythentication failures' :
 **PAM** acted as a "gatekeeper," counting the failures.
* 'Connection closed by authenticating user q [preauth]' : After 3 failures, the server cut the conection to prevent a **Brute-Force attack**.

### 4. case B : Successful Login
After being kicked out, I reconnected and entered the correct password.

**observed Log Results**
* 'Accected password for q from 192.168.64.1': The system confirmed the identity.
* 'pam_unix(sshd:session) : session opened fir user q' : The gatekeeper opened the door and started a new session.

---

### 5. Conclusion
Linux is incredibly transparent. Every action, especially security-related ones, is recorded in '/var/log/auth.log'. By monitoring these logs,
a security analyst can detect and block intruders in real-time.


