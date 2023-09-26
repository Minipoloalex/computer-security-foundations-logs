
# Trabalho realizado nas Semanas #2 e #3

## CVE-2010-2568

### Identification:
- This vulnerability in Windows systems allowed for remote code execution through certain files
- It comes from improper input validation by the operating system
- The vulnerability existed in Microsoft Windows XP SP3, Server 2003 SP2, Vista SP1 and SP2, Server 2008 SP2 and R2, and Windows 7
- This vulnerability was one of many vulnerabilities exploited in the Stuxnet attack


### Cataloguing:
- This vulnerability was officially reported in June 2010
- It is considered very critical, with high impact on confidentiality, integrity and availability (CVSS score of 9.3)
- It is a case of the common weakness CWE-20, improper input validation of an icon, more specifically CWE-1287, improper validation of specified type of input.
- The governments of USA and Israel are suspected of having exploited this vulnerability, targeting Iran

### Exploit:
- Exploited via a crafted shorcut file (.LNK) or a program information file (.PIF)
- The icon of the file points to a malicious DLL (Dynamic-Link Library)
- When the operating system tries to render the file's icon, the malware is stealthly executed
- Metasploit module: Microsoft Windows Shell LNK Code Execution

### Attacks:
- The Stuxnet malware, thought to have been developed by U.S. and Israeli intelligence, exploits this vulnerability
- Led to the significant disruption of Iranian nuclear facilities, in 2010
- Raised awareness to the potential damage of cyber attacks in critical infrastructure systems
- Exists at least one known exploit to this vulnerability!
