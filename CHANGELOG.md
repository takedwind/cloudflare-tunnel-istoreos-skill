# Changelog

## [1.0.0] - $(date +%Y-%m-%d)
### Added
- **Complete End-to-End Automation:** The skill now covers the entire process from downloading the installation package to starting the service.
- **Dynamic Architecture Detection:** Automatically detects the router's architecture (e.g., x86_64, aarch64) and downloads the compatible `.ipk` package.
- **Automated Installation & Config:** Automatically installs the `cloudflared` package, manages UCI configuration, and writes the user token.
- **Dual-Language Documentation:** Instructions in `SKILL.md` and `README.md` are now fully bilingual (English and Chinese).
- **Mermaid Flowchart:** Added a visual flowchart outlining the Agent's automated execution path.
- **Detailed File Documentation:** Expanded documentation on core configuration paths (`/etc/config/cloudflared`) and log troubleshooting.
- **Troubleshooting Guide:** Included specific steps to bypass Cloudflare TLS validation errors (HTTP 503) for local OpenWRT web interfaces.
