# OWASP Top 10

## Injections
* An injection of code happens when an attacker sends invalid data to the web application with the intention to make it do something different from what the application was designed/programmed to do.
* When untrusted data is sent to an interpreter as part of a command or query. The attacker’s hostile data can trick the interpreter into executing unintended commands or accessing data without proper authorization.
### Solutions
* Query Parameterization
* Validation and sanitization

## Broken Authentication
* Application functions related to authentication and session management are often implemented incorrectly, allowing attackers to compromise passwords, keys, or session tokens, or to exploit other implementation flaws to assume other users’ identities temporarily or permanently.
### Attack Attempts
* [Credential stuffing](https://owasp.org/www-community/attacks/Credential_stuffing)
* Automated attacks
* Default passwords/user names
### Solutions
* Multi factor auth
* Password checking: Check if the password is too common to use. 
* Password complexity
* Limit failed login
* Server-side session management

## Sensitive Data Exposure
* Attackers may steal or modify such weakly protected data to conduct credit card fraud, identity theft, or other crimes. 
* Sensitive data may be compromised without extra protection, such as encryption at rest or in transit, and requires special precautions when exchanged with the browser.
### Issues and Attack Attempts
* TLS with week ciphers(?)
* Rainbow table
### Solutions
* Classify data
* Apply controls to different kinds of data
* Encrypt data at rest(with strong encryption)
* Strong ciphers
* Possible PFS (Perfect Forward Secrecy) and/or HSTS(HTTP Strict Transport Security)

## XML External Entities (XXE)
* Many older or poorly configured XML processors evaluate external entity references within XML documents. 
* External entities can be used to disclose internal files using the file URI handler, internal file shares, internal port scanning, remote code execution, and denial of service attacks.
### Attack Attempts
* Billion Laughs

### Solutions

* Config XML parser properly. Disable XXE completely if you can.
* Input validation
* Source code analysis tool
* Dynamic application security testing

## Broken Access Control

* Restrictions on what authenticated users are allowed to do are often not properly enforced.
* Attackers can exploit these flaws to access unauthorized functionality and/or data, such as access other users’
  accounts, view sensitive files, modify other users’ data, change access rights, etc.

#### Access Control Challenges

* `Access Control is difficult to test from automated tools`. Your scanning tools are rarely aware of your custom access
  control policies.
* Access Control is difficult for developers to build. Our frameworks rarely provide detailed access control
  functionality.

#### Best Practice: Code to the Activity (or Permission)

* Code it once, never needs to change again
* Implies policy is centralized in some way
* Implies policy is persisted in some way
* Requires more design/work up front to get right

#### Access Control Key Concepts

* Enforce access control by an activity or feature, not the role
* Implement data-contextual access control to assign permissions to application users in the context of specific data
  items for horizontal access control requirements
* Build a centralized access control mechanism
* Design access control so all requests must be authorized
* Deny by default, fail securely
* Server-side trusted data should drive access control policy decisions
* Be able to change a users entitlements in real time
* Build grouping capability for users and permissions
* Build admin screens first to manage access control policy data

### CAUTION

* Good access control is `hard to add to an application late in the lifecycle`.

### VERIFY

* Automated security tools are poor at verifying access control vulnerabilities since tools are not aware of your access
  control policy

### GUIDANCE

* https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html
* http://nvlpubs.nist.gov/nistpubs/specialpublications/NIST.sp.800-162.pdf
* https://github.com/OWASP/ASVS/blob/master/4.0/en/0x12-V4-Access-Control.md

### Solutions

* Dynamic application security testing tools
* Static application security testing tools/source code analysis tool
* Manual testing
* Only give minimal access for that one thing for the minimum amount of time.

## Security Misconfiguration

* Security misconfiguration is the most commonly seen issue. This is commonly a result of insecure default
  configurations, incomplete or ad hoc configurations, open cloud storage, misconfigured HTTP headers, and verbose error
  messages containing sensitive information.
* Not only must all operating systems, frameworks, libraries, and applications be securely configured, but they must be
  patched/upgraded in a timely fashion.
### Solutions
* Repeatable hardening process: test these process with automation tools.  
* All servers - same config
* Minimal platform: Avoid unnecessary features(more feature -> more vulnerabilities). Don't use what you don't need -> minimize potential risk.
* Security directives: HSTS, HPKP, X-Frame-Options
* Don't over sharing information. (e.g. Error handling disclose the version of the Tomcat you use.)

## Cross-Site Scripting XSS 
* Client-side code injection
* XSS flaws occur whenever an application includes untrusted data in a new web page without proper validation or escaping, or updates an existing web page with user-supplied data using a browser API that can create HTML or JavaScript. 
* XSS allows attackers to execute scripts in the victim’s browser which can hijack user sessions, deface web sites, or redirect the user to malicious sites.
### Solutions
* Use modern browsers which inherently protect against XSS.
* Separate untrusted data(any kind of user input data) from active browser content.(?)
* Encode and sanitize

## Insecure Deserialization 
* Insecure deserialization often leads to remote code execution. 
* Even if deserialization flaws do not result in remote code execution, they can be used to perform attacks, including replay attacks, injection attacks, and privilege escalation attacks.
### Solutions 
* validate any kinds of user input.

## Using Components with Known Vulnerabilities 
* Components, such as libraries, frameworks, and other software modules, run with the same privileges as the application. If a vulnerable component is exploited, such an attack can facilitate serious data loss or server takeover. 
* Applications and APIs using components with known vulnerabilities may undermine application defenses and enable various attacks and impacts.
### Solutions
* Continuous inventory of clients and servers
* Downloads components from trusted sources.
* Plan, monitor, patch, config
* Common vulnerabilities and exploits(CVE), National vulnerabilities database(NVD)

## Insufficient Logging & Monitoring
* Insufficient logging and monitoring, coupled with missing or ineffective integration with incident response, allows attackers to further attack systems, maintain persistence, pivot to more systems, and tamper, extract, or destroy data. 
* Most breach studies show time to detect a breach is over 200 days, typically detected by external parties rather than internal processes or monitoring.
### Solutions
* Sufficient content
* Good format
* Integrity controls
* response plan

# References
* [OWASP Top Ten](https://owasp.org/www-project-top-ten/)
* [OWASP Top Ten - 2017](https://www.youtube.com/playlist?list=PLyqga7AXMtPPuibxp1N0TdyDrKwP9H_jD)
* [SSL](https://youtu.be/33VYnE7Bzpk)
* [Jim Manico - OWASP Top Ten 2021](https://youtu.be/pLsH-TT26Mo)



