ImpactCheck
Prevent risky configuration changes before they break production.
ImpactCheck is a Windows CLI tool that detects high-risk configuration changes (JSON, XML, .config) and system DLL impact before deployment.
Instead of discovering problems in staging or production, you detect them in CI.

Why It Exists
Most CI pipelines validate syntax.
They do not validate impact.
A one-line change in:
a token
a connection string
an endpoint
a feature flag
can silently change system behavior.

ImpactCheck makes configuration changes measurable and enforceable.
Quick Example
impactcheck json appsettings.json --snapshot before.json
# modify file
impactcheck json appsettings.json --snapshot after.json
impactcheck diff before.json after.json --policy strict

If a sensitive value changes, the exit code becomes:
1

Your pipeline fails before deployment.
ImpactCheck
Make configuration changes measurable before they break production.
ImpactCheck is a cross-platform CLI tool that analyzes configuration and binary changes and tells you:
What changed
What is risky
Whether the change should be allowed
It is designed to run locally and inside CI/CD pipelines as an automated safety gate.

🚩 Why ImpactCheck
Configuration changes are one of the most common causes of production incidents.
A small modification in:
a token
a connection string
an endpoint
a system DLL
can silently introduce high-risk behavior.
ImpactCheck makes configuration changes explicit, comparable, and enforceable.

🔍 What It Does
ImpactCheck can:
Analyze JSON
Analyze XML
Analyze .config files
Analyze DLLs
Generate snapshots
Compute diffs
Enforce policies via exit codes
Output human-readable reports or JSON
It works fully offline and does not require cloud services.

📦 Installation
Windows (Recommended)

Download the latest release package, which includes:
impactcheck.exe
install.ps1
uninstall.ps1

Install (System-wide – recommended)
Open PowerShell as Administrator and run:
powershell -ExecutionPolicy Bypass -File .\install.ps1

This will:
Copy impactcheck.exe to
C:\Program Files\ImpactCheck\

Add the installation directory to your system PATH
Make impactcheck available globally

Install (User-only – no admin required)
powershell -ExecutionPolicy Bypass -File .\install.ps1 -UserOnly
This installs ImpactCheck under:
%LOCALAPPDATA%\ImpactCheck\
and updates the user PATH.

⚠ Important
After installation, you must:
Close and reopen PowerShell or CMD or Restart your terminal session
This ensures the updated PATH is loaded correctly.

Verify Installation
Open a new terminal and run:
impactcheck
If installed correctly, the help message will be displayed.

Uninstall
To remove ImpactCheck:
System-wide:
powershell -ExecutionPolicy Bypass -File .\uninstall.ps1

User-only:
powershell -ExecutionPolicy Bypass -File .\uninstall.ps1 -UserOnly

🚀 Quick Start
Analyze JSON
impactcheck json appsettings.json

Analyze XML
impactcheck xml test.xml

Analyze .config
impactcheck config App.config
config is intended for .config, app.config, web.config, .exe.config, and similar files.

Analyze DLL
impactcheck dll C:\Windows\System32\sechost.dll

📸 Snapshots
Create a snapshot of the current state:
impactcheck json appsettings.json --snapshot before.json
Modify the file, then create another snapshot:
impactcheck json appsettings.json --snapshot after.json
🔄 Diff
Compare two snapshots:
impactcheck diff before.json after.json

Example output:
~ Changed findings (1):
  ~ JSON_KEY|Api:Token
     Value: CHANGED
JSON diff output
impactcheck diff before.json after.json --json

🛑 Policies
ImpactCheck can enforce policies during diff.
Strict Policy Example
impactcheck diff before.json after.json --policy strict
echo $LASTEXITCODE

Exit codes:
Code	Meaning
0	OK
1	Policy violated
2	Uncertain
3	Error

This makes ImpactCheck ideal for CI/CD environments.
🔧 CI/CD Example
GitHub Actions
name: Config Safety
on: [push]

jobs:
  impactcheck:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run ImpactCheck
        run: |
          impactcheck diff before.json after.json --policy strict
If ImpactCheck exits with 1, the pipeline fails.

📁 Supported Commands
Command	Description
dll	Analyze a DLL
json Analyze a JSON file
xml	Analyze a generic XML file
config	Analyze configuration-style XML
diff	Compare two snapshots
🧠 Design Philosophy

ImpactCheck does not lint, format, or modify files.

It focuses strictly on:

Change → Impact → Risk → Decision

The goal is not to style configuration.
The goal is to prevent production incidents.

🛣 Roadmap
Custom policy configuration files
Rule packs for .NET / IIS / DevOps
YAML support
Signed binaries
MSI installer
CI integrations with PR summaries
Enterprise policy packs

🛡 What ImpactCheck Is Not
Not a formatter
Not a linter
Not a cloud service
Not AI-driven
It is a deterministic, transparent change-impact engine.


1) Human output vs JSON output (--json)

Human-readable (default):

impactcheck json appsettings.json


Machine-readable JSON (for scripts/CI):

impactcheck json appsettings.json --json

2) Creating snapshots (--snapshot) and comparing them (diff)
Create a snapshot:
impactcheck json appsettings.json --snapshot before.json

Modify the file, then:
impactcheck json appsettings.json --snapshot after.json


Compare:
impactcheck diff before.json after.json

Machine-readable diff:
impactcheck diff before.json after.json --json
3) Blocking risky changes with policies (--policy strict)
impactcheck diff before.json after.json --policy strict
echo $LASTEXITCODE
0 → OK
1 → policy violated (block deployment)
This is the recommended way to use ImpactCheck in CI/CD.

4) JSON example: Secret + Endpoint detection
appsettings.json:
{
  "Api": {
    "BaseUrl": "https://api.example.com",
    "Token": "abc123"
  }
}
Run:
impactcheck json appsettings.json

Expected behavior:
Api:BaseUrl flagged as NETWORK_ENDPOINT
Api:Token flagged as SECRET
status becomes NOT_ISOLATED (review recommended)

5) XML / .config example: Connection string detection
App.config:
<configuration>
  <connectionStrings>
    <add name="Main"
         connectionString="Server=.;Database=Prod;User Id=sa;Password=SuperSecret"/>
  </connectionStrings>
</configuration>

Run:
impactcheck config App.config

Expected behavior:
detects /configuration/connectionStrings/add/@connectionString
flags it as CONNECTION_STRING
status becomes NOT_ISOLATED
🧩 DLL Examples (what ImpactCheck actually does)
DLL analysis answers a different question than configs:
“Is this DLL in use right now, and by what processes/services?”
6) DLL impact scan (human output)
impactcheck dll C:\Windows\System32\sechost.dll

ImpactCheck will:
enumerate running processes
detect which processes have the DLL loaded
resolve the user owning the process (when available)
map processes to Windows Services (Service Control Manager)
flag risk based on where the DLL is used (system services are high-impact)
Typical findings:
svchost.exe instances loading the DLL
each instance linked to one or more Windows services
“risk” flags if the DLL is used by critical services

7) DLL scan as JSON (automation)
impactcheck dll C:\Windows\System32\sechost.dll --json
This is useful when you want to:
log results
store snapshots
compare later
ingest into other tools

8) DLL snapshots + diff (before/after)
This is useful after:
patching
replacing a DLL
changing dependencies
enabling/disabling services
impactcheck dll C:\Windows\System32\sechost.dll --snapshot before.json
# apply a change (patch/update/service change)
impactcheck dll C:\Windows\System32\sechost.dll --snapshot after.json
impactcheck diff before.json after.json

Expected behavior:
shows which processes/services appeared/disappeared
shows changes in service associations
helps detect unintended blast radius

✅ Quick “one-liner” examples
Config gating in CI:
impactcheck diff before.json after.json --policy strict

Generate JSON output for a report:
impactcheck json appsettings.json --json > report.json

Compare snapshots in automation:
impactcheck diff before.json after.json --json > diff.json

📄 License
opyright (c) 2025 Federico
All rights reserved.
This software may not be copied, modified, distriuted, sublicensed, or sold without explicit written permission..
Created By Federico Distaso

Enterprise Edition (Coming Soon)
ImpactCheck Enterprise will include:
Custom policy files (fine-grained rule control)
Organization-wide rule packs
CI pull request comments
Compliance reports
Audit logging
Team licensing

If you are evaluating ImpactCheck for production use,
please reach out at: *fededistaso@gmail.com*
