# react2shell-scanner

A command-line tool for detecting CVE-2025-55182 and CVE-2025-66478 in Next.js applications using React Server Components.

For technical details on the vulnerability and detection methodology, see our blog post: https://slcyber.io/research-center/high-fidelity-detection-mechanism-for-rsc-next-js-rce-cve-2025-55182-cve-2025-66478

## How It Works

This scanner implements a high-fidelity detection mechanism for CVE-2025-55182 and CVE-2025-66478, which are critical unauthenticated remote code execution (RCE) vulnerabilities in React Server Components.

### Detection Method

The scanner sends a specially crafted multipart POST request to the target application with:
- **Custom Headers**: Including `Next-Action`, `X-Nextjs-Request-Id`, and `Next-Router-State-Tree`
- **Multipart Payload**: Two form-data parts containing specific values that trigger the deserialization vulnerability

### Vulnerability Signature

Vulnerable hosts exhibit a specific response pattern:
1. **HTTP Status Code 500** (Internal Server Error)
2. **Response body contains** `E{"digest"` - a unique error signature from React's Flight protocol deserialization

This high-fidelity approach differentiates truly vulnerable hosts from:
- Hosts that are simply running RSC but are patched
- Generic 500 errors from other causes
- Non-RSC applications

### Why This Works

The vulnerability stems from unsafe deserialization in React's "Flight" protocol used by React Server Components. The crafted payload exploits how vulnerable versions handle multipart form data, triggering a specific error path that exposes the digest marker only in unpatched versions.

## Requirements

- Python 3.9+
- requests
- tqdm

## Installation

```
pip install -r requirements.txt
```

## Usage

Scan a single host:

```
python3 scanner.py -u https://example.com
```

Scan a list of hosts:

```
python3 scanner.py -l hosts.txt
```

Scan with multiple threads and save results:

```
python3 scanner.py -l hosts.txt -t 20 -o results.json
```

## Options

```
-u, --url         Single URL to check
-l, --list        File containing hosts (one per line)
-t, --threads     Number of concurrent threads (default: 10)
--timeout         Request timeout in seconds (default: 10)
-o, --output      Output file for results (JSON)
--all-results     Save all results, not just vulnerable hosts
-k, --insecure    Disable SSL certificate verification
-v, --verbose     Show response details for vulnerable hosts
-q, --quiet       Only output vulnerable hosts
--no-color        Disable colored output
```

## Output

Results are printed to the terminal. When using `-o`, vulnerable hosts are saved to a JSON file containing the full HTTP request and response for verification.

## Affected Versions

### React Server Components (CVE-2025-55182)
**Vulnerable versions:**
- React 19.0.0, 19.1.0, 19.1.1, 19.2.0
- react-server-dom-webpack (all 19.x versions before patch)
- react-server-dom-parcel (all 19.x versions before patch)
- react-server-dom-turbopack (all 19.x versions before patch)

**Patched versions:**
- React 19.0.1, 19.1.2, 19.2.1 and later

### Next.js (CVE-2025-66478)
**Vulnerable versions:**
- Next.js 14.3.0-canary.77 and later canary releases
- Next.js 15.x (all unpatched versions)
- Next.js 16.x (all unpatched versions)

**Patched versions:**
- Next.js 15.0.5, 15.1.9, 15.2.6, 15.3.6, 15.4.8, 15.5.7
- Next.js 16.0.7 and later

### Other Affected Frameworks
- React Router (using unstable RSC APIs)
- Waku
- Expo (using RSC)
- Redwood SDK
- @vitejs/plugin-rsc
- @parcel/rsc

## Important Notes

- **High-Fidelity Detection**: This scanner uses the official detection method from Assetnote Security Research Team to minimize false positives
- **No Exploitation**: The scanner only detects the vulnerability and does not exploit it
- **Immediate Patching**: If vulnerabilities are found, upgrade immediately to patched versions
- **CVSS Score**: Both CVEs are rated 10.0 (Critical) - maximum severity
