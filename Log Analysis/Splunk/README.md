Overview

This report details the investigation of a potential security breach in an AWS environment. It covers the analysis of log files, identification of malicious activity, the tools used during the investigation, and security recommendations to prevent future incidents. The findings highlight the need for stronger security measures to ensure the protection of sensitive data and infrastructure.

Tools Used

- Wireshark: For analyzing the PCAP file and inspecting network traffic.
- Splunk: For aggregating and visualizing the log data to detect anomalous patterns and correlate events across multiple logs.

Investigation Process

1\. Initial Analysis with Wireshark:

- Filtered by HTTP Protocol: Narrowed the focus to relevant traffic.
- Identified large packets: Targeted packets over 600 MB to locate potential data downloads.
- Tracked TCP and HTTP Streams: Enabled clearer visualization of data, revealing:
  - Stolen packets containing ARNs and IDs tied to compromised credentials.

2\. Detailed Analysis with Splunk:

- Filtered logs using arn:aws:iam: Isolated entries connected to the stolen keys.
- Applied additional filters for webapp-role: Focused on the compromised role, uncovering:
  - GET and HEAD requests for sensitive files, such as SSH keys and password files.

Investigation Findings

1\. Has the attacker managed to compromise the environment?

Yes, the environment was compromised. The attacker gained unauthorized access by exploiting the AWS metadata service to retrieve temporary IAM credentials. These credentials were then used to assume the webapp-role and gain access to AWS resources, including S3 buckets and EC2 instances.

Evidence:

- Suspicious IP (18.192.239.172) made API calls and participated in potential data exfiltration.
- Metadata service logs revealed GET requests to <http://169.254.169.254>, successfully retrieving credentials tied to the webapp-role.
- S3 logs confirmed GET and HEAD requests for SSH keys (id_ed25519, id_rsa).
- HTTP headers indicated the use of automation scripts, such as Ruby (User-Agent: Ruby) and curl (User-Agent: curl/7.61.1).

2\. What techniques were used by the attacker?

The following techniques, mapped to MITRE ATT&CK, were identified:

- T1552.001 - Unsecured Credentials in Metadata: Exploited the AWS metadata service to retrieve IAM credentials.
- T1071.001 - Application Layer Protocol (HTTP/S): Used HTTP/S to interact with AWS APIs and S3 buckets for data exfiltration.
- T1003 - Credential Dumping: Compromised temporary IAM credentials (AccessKeyId, SecretAccessKey) to assume the webapp-role.

3\. Has the attacker been able to access sensitive data (such as customer data or credentials)?

Yes, the attacker accessed sensitive data, including SSH keys (id_ed25519, id_rsa) critical for accessing systems. Other potential files (e.g., passwords) require deeper analysis.

Evidence:

- Retrieved credentials: AccessKeyId, SecretAccessKey, and Token tied to arn:aws:iam::486288459312:instance-profile/webapp-role.
- S3 logs show successful downloads of SSH key files.

4\. How would you handle this incident?

Immediate Actions:

- Revoke compromised IAM credentials and block suspicious IPs (e.g., 18.192.239.172).
- Isolate affected AWS resources and reset passwords/tokens for impacted accounts.

Recovery Actions:

- Audit CloudTrail logs to trace activity performed with the compromised role.
- Restore systems and data from secure backups.
- Enforce least privilege access policies and conduct forensic reviews for lingering threats.

Communication:

- Notify internal teams and comply with reporting obligations to affected customers and authorities.

5\. What security recommendations and improvements would you propose to avoid this kind of incident from happening again?

- Identity & Access Management: Enforce IMDSv2 and Multi-Factor Authentication (MFA).
- Monitoring & Detection: Enable CloudTrail, GuardDuty, and alerts for unauthorized access.
- Network Security: Use VPC Flow Logs and AWS Network Firewall to restrict sensitive traffic.
- Encryption: Encrypt sensitive data at rest and in transit using AWS KMS.
- Awareness & Training: Train employees on best practices and conduct regular penetration testing.

Impact Analysis

The security breach had significant implications for the AWS environment and the organization as a whole:

1. Compromised Credentials: Temporary IAM credentials (AccessKeyId, SecretAccessKey, Token) were misused to assume the webapp-role and access AWS resources.
2. Sensitive Data Exposure: SSH keys were exposed, presenting risks of unauthorized access.
3. Data Exfiltration: Logs revealed downloads of sensitive files, raising confidentiality concerns.
4. Operational and Financial Implications: Potential downtime and disruptions due to credential revocation and system isolation. Compliance challenges and legal ramifications requiring formal reporting.
5. Reputational Damage: Potential loss of trust among customers and stakeholders.

Conclusion

This investigation revealed a critical vulnerability involving the AWS metadata service, leading to compromised IAM credentials. Implementing stronger IAM policies, real-time monitoring, network security measures, and training programs will mitigate risks and reduce the likelihood of similar breaches.
