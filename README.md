# Cyberster Internship 2026

## Week 1 – Internship Progress

### Overview
This section summarizes the work completed during **Week 1** of the Cyberster Internship 2026.

### Objectives
- Get familiar with the project structure and development workflow.
- Set up the local development environment.
- Understand repository guidelines and coding standards.
- Complete initial onboarding and starter tasks.

### Tasks Completed
- Cloned the repository and explored folder structure.
- Configured required tools and dependencies.
- Verified project setup by running/building the project.
- Reviewed contribution workflow (branches, commits, pull requests).
- Completed assigned starter task(s).

### Learnings
- Better understanding of project architecture.
- Improved Git/GitHub collaboration workflow.
- Practical exposure to team development standards.

### Challenges Faced
- Initial environment/dependency setup issues.
- Understanding unfamiliar modules and code conventions.

### How Challenges Were Resolved
- Referred to project documentation and setup notes.
- Sought clarification from mentors/teammates.
- Performed troubleshooting with logs and dependency checks.

### Deliverables
- Local environment setup completed.
- Initial code/task submission done.
- Week 1 progress documentation integrated in root README.

### Plan for Week 2
- Continue with assigned development tasks.
- Focus on deeper module-level understanding.
- Contribute to feature improvements and bug fixes.

## Week 2 – Internship Progress

### Overview
This section summarizes the work completed during **Week 2** of the Cyberster Internship 2026.

### Objectives
- Perform advanced network scanning and host/service discovery.
- Identify running services and gather accurate version information.
- Use Nmap scripting and manual enumeration for vulnerability insights.
- Practice stealth/evasion scan techniques in a controlled lab.

### Tasks Completed
- Completed TCP Connect and SYN Stealth full-port scans.
- Performed UDP top-ports scanning and targeted UDP checks.
- Executed timing template comparisons (T0–T5) and analyzed speed differences.
- Conducted service fingerprinting with `-sV` and aggressive scan with `-A`.
- Performed manual banner grabbing using `nc`/`telnet`.
- Enumerated SMB services using `enum4linux`/`smbclient`.
- Ran NSE scripts for discovery, safe checks, and vulnerability detection.
- Mapped discovered services/versions to likely CVEs.
- Practiced evasion flags (`-f`, decoys, source-port spoofing) and documented behavior.

### Learnings
- Clear understanding of differences between scan types (`-sT`, `-sS`, `-sU`).
- Improved ability to correlate service banners with potential vulnerabilities.
- Hands-on experience with NSE categories and targeted scripts.
- Better understanding of practical limitations of stealth techniques in default lab setups.

### Challenges Faced
- Long scan times for full-port/low-timing template scans.
- Interpreting filtered/open|filtered UDP results accurately.
- Distinguishing between syntax demonstration and real-world evasion effectiveness.

### How Challenges Were Resolved
- Used focused port ranges where appropriate and documented timing trade-offs.
- Cross-validated findings with multiple commands/tools.
- Referenced NVD/Vulners/Searchsploit to verify vulnerability mapping.
- Documented environment constraints (no default IDS on Metasploitable2).

### Deliverables
- Nmap output files/screenshots for Task 01–03.
- Service table with ports, service names, versions, and potential vulnerabilities.
- Evasion write-up for Task 04 with observations.
- Week 2 progress documentation integrated in root README.

### Plan for Week 3
- Validate high-priority findings through safe proof-of-concept steps.
- Improve reporting quality with remediation-focused recommendations.
- Begin deeper exploitation workflow in a controlled and authorized environment.
