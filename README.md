# Cybersecurity & SOC Internship Reports

**Kıvanç Bergama** — Computer Engineering Student, Eskişehir Osmangazi University

---

## 🇹🇷 Türkçe

Yaz dönemi staj sürecinde Wazuh SIEM/XDR platformu üzerinde gerçekleştirdiğim uygulamalı SOC (Security Operations Center) çalışmalarının haftalık raporları. Her rapor Türkçe ve İngilizce olarak hazırlanmıştır.

### 🛠️ Kullanılan Teknolojiler
- **SIEM/XDR:** Wazuh (Manager, Indexer, Dashboard, Agent)
- **Ortam:** Docker, WSL2 (Ubuntu 24.04 LTS), Windows 11
- **Uç Nokta İzleme:** Sysmon, File Integrity Monitoring (FIM)
- **Tehdit İstihbaratı:** VirusTotal Public API v3
- **Otomasyon:** Python 3 (Wazuh REST API), Slack Incoming Webhooks (SOAR)
- **Saldırı Simülasyonu:** Hydra, MITRE ATT&CK framework
- **Uyumluluk:** PCI DSS, GDPR eşleştirmesi

### 📁 İçerik
| Hafta | Konu |
|---|---|
| [Hafta_01](./Hafta_01) | SOC lab kurulumu, SSH brute-force simülasyonu, ajan sağlık kontrolü, syslog yapılandırması, zafiyet (CVE) tespiti |
| [Week_02](./Week_02) | File Integrity Monitoring (FIM) yapılandırması, özel kural (custom rule) geliştirme |
| [Week_03](./Week_03) | Sysmon ile derinlemesine Windows analizi, VirusTotal CTI entegrasyonu, Slack SOAR otomasyonu, Python REST API betiği |

### 🎯 Öne Çıkanlar
- MITRE ATT&CK tekniklerine (T1110 Brute Force, T1203, T1078 vb.) eşlenen gerçek zamanlı tehdit tespiti
- Sıfırdan özel Wazuh kuralı (custom rule) yazımı ve XML parser sorunlarının çözümü
- VirusTotal API ile otomatik zararlı dosya tespiti ve Slack üzerinden anlık uyarı (SOAR)
- PCI DSS ve GDPR uyumluluk gereksinimleriyle güvenlik olaylarının ilişkilendirilmesi
- Wazuh REST API üzerinden ajan durumlarının Python ile programatik sorgulanması

> **Not:** Tüm çalışmalar izole, kişisel bir laboratuvar ortamında (WSL2/Docker) gerçekleştirilmiştir. Raporlarda yer alan IP adresleri ve API anahtarları test/placeholder değerleridir.

---

## 🇬🇧 English

Weekly reports from my summer internship, covering hands-on SOC (Security Operations Center) work on the Wazuh SIEM/XDR platform. Each report is written in both Turkish and English.

### 🛠️ Tech Stack
- **SIEM/XDR:** Wazuh (Manager, Indexer, Dashboard, Agent)
- **Environment:** Docker, WSL2 (Ubuntu 24.04 LTS), Windows 11
- **Endpoint Monitoring:** Sysmon, File Integrity Monitoring (FIM)
- **Cyber Threat Intelligence:** VirusTotal Public API v3
- **Automation:** Python 3 (Wazuh REST API), Slack Incoming Webhooks (SOAR)
- **Attack Simulation:** Hydra, MITRE ATT&CK framework
- **Compliance:** PCI DSS, GDPR mapping

### 📁 Contents
| Week | Topic |
|---|---|
| [Hafta_01](./Hafta_01) | Lab setup, SSH brute-force simulation, agent health checks, syslog configuration, vulnerability (CVE) detection |
| [Week_02](./Week_02) | File Integrity Monitoring (FIM) configuration, custom rule development |
| [Week_03](./Week_03) | Deep Windows analysis with Sysmon, VirusTotal CTI integration, Slack SOAR automation, Python REST API scripting |

### 🎯 Highlights
- Real-time threat detection mapped to MITRE ATT&CK techniques (T1110 Brute Force, T1203, T1078, etc.)
- Custom Wazuh rule development from scratch, including resolving XML parser issues
- Automated malicious file detection via VirusTotal API with instant Slack alerting (SOAR)
- Correlating security incidents with PCI DSS and GDPR compliance requirements
- Programmatic agent status querying via the Wazuh REST API using Python

> **Note:** All work was performed in an isolated, personal lab environment (WSL2/Docker). IP addresses and API keys shown in the reports are test/placeholder values.

---

📫 Sorularınız için / For questions, feel free to reach out via GitHub.
