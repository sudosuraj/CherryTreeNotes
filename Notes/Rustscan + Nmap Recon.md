# Rustscan + Nmap Recon

Fast full-port scan piped into Nmap for service/version detection.

```bash
rustscan -b 500 -a <TARGET> --top -- -sC -sV -Pn
```

Batch size `-b`: 500 trades speed for stability, 1500 is the default balance, higher risks missed/unstable responses.
