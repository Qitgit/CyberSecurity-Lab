### Overview

The Session Layer is Layer 5 of the OSI model. It's responsible for establishing, managing, and terminating **sessions** - a session being a persistent,
logical conversation between two applications that may involve multiple exchanges of data over time, not just a single request-response.

Key responsibilities of this layer:

-**Session Establishment**: setting up a "conversation" context between two applications (distinct from a TCP connection, which is Layer 4)
-**Session Maintenance**: keeping track of where the conversation is up to - useful for long-running exchanges like a login session or a file transfer
-**Synchronization**: inserting checkpoints into a data stream, so if a sessions is interrupted, it can resume from the last checkpoint instead of starting over
-**Session Termination**: gracefully closing the session when the conversation is complete


### Why This Layer Is Often "Invisible" in Practice

Unlike Layers 1-4, the Session Layer doesn't have a single obvious protocol you can filter for in Wireshark the way you can filter 'TCP' or 'ARP'.
In modern networking, session management responsibilities are largely absorbed into :

- **Application-layer mechanisms**: HTTP cookies, session tokens, and authentication states (e.g., staying logged into a website) - these are technically Layer 7 constructs
- that fulfill what was originally envisioned as a Layer 5 job.

  -**TLS session resumption**: TLS itself maintains session state to avoid repeating a full handshake on reconnect
  -Older protocols like **NetBIOS**,**RPC**,ad **SMB**(Windows file sharing) do have explicit session-later concepts baked in

  This is a well-known criticism of the OSI model: real-world protocol stacks(like TCP/IP) don't cleanly separate Layers 5-7 the way OSI theoretically describes them.
  Most of what OSI calls "Session Layer" work today happens inside application-layer protocols.



<img width="187" height="45" alt="Screenshot 2026-08-05 173427" src="https://github.com/user-attachments/assets/819f2104-3477-42fe-ba4c-ce2d109326e9" />


<img width="878" height="361" alt="Screenshot 2026-08-05 173449" src="https://github.com/user-attachments/assets/93dce5ee-33df-4801-b22e-ddd6bf225fea" />

  ## Security Perspective

  ###  Session Hijacking

  An attacker steals or predict a vaild session token and uses it to impersonate the legitimate user - without needing their password.

  **How the token can be stolen**
  -**XSS (Cross-Site Scripting)**: malicious JavaScript injected into a page reads 'document.cookie' and exfiltrates the session value - mitigated by the **HttpOnly** flag, which blocks
  JavaScript from accessing the cookie at all.
  -**Network sniffing**: capturing the session token in transit over an unencrypted connection - mitigated by the **Secure**flag, which ensures the cookie is only evet sent over HTTPS
  -**Session token leakage**: accidentally exposing the token in logs, URLs, error messages, or screenshots (this is why cookie *Values* should never be shown when documenting a lab like this one)

**Relevant cookies from this capture**

  **Relevant cookies from this capture**:
  -'user_session', '__Host-user-session-same-site' : there carry the actual authenticated session state. If leaked, an attacker with this value alone can potentially act as the logged-in user.
  - `user_session`, `__Host-user-session-same-site`: these carry the actual 
  authenticated session state. If leaked, an attacker with this value alone can 
  potentially act as the logged-in user
- The `__Host-` prefix is a browser-enforced security mechanism — GitHub cannot 
  set this cookie unless it also sets `Secure=true`, `Path=/`, and omits the 
  `Domain` attribute. This prevents subdomain-based cookie injection attacks, 
  where a compromised subdomain could otherwise overwrite or plant a cookie 
  readable by the main domain

**Detection**: the same session token being used from two 
geographically distant IP addresses within an unrealistic time window, or a 
sudden User-Agent change mid-session, are common indicators used in SIEM 
correlation rules to flag possible session hijacking.

---

### Session Fixation

A related but distinct attack: instead of stealing an existing session, the 
attacker **forces a known session ID onto the victim** before they log in.

**Attack flow**
1. Attacker obtains a valid (but unauthenticated) session ID from the target site
2. Attacker tricks the victim into using that same session ID (e.g., via a 
   crafted link)
3. Victim logs in normally — but the session ID stays the same, now becoming 
   *authenticated*
4. Attacker, already knowing that session ID, is now also logged in as the victim

**Mitigation:** regenerating the session ID immediately after a successful login 
(rather than reusing the pre-login session ID) — a standard practice in secure 
session management.

---

###  Classifying Cookies by Security Relevance

Not every cookie carries the same risk if exposed. From this capture:

| Security-Critical (session/auth) | Low-Risk (preferences/analytics) |
|---|---|
| `user_session` | `color_mode` / `preferred_color_mode` |
| `__Host-user-session-same-site` | `tz` (timezone) |
| `_gh_sess` | `MSFPC` (Microsoft tracking) |
| `dotcom_user` (likely contains username) | `MicrosoftApplicationsTelemetry...` |
| `logged_in` | `cpu_bucket` (A/B testing) |
| `saved_user_sessions` | |
| `datadome` (bot-detection token) | |

**Why this distinction matters for a SOC analyst:** during incident response or 
log review, not all cookie/session activity is equally significant — knowing 
which cookies represent actual authentication state (and therefore warrant close 
monitoring) versus which are harmless UI/analytics data helps prioritize 
investigation effort and avoid alert fatigue from noise.
