# SİBER GÜVENLİK VE SIEM/XDR LABORATUVAR PROJESİ RAPORU

**Konu:** Wazuh SIEM/XDR Platformu Üzerinde CTI (VirusTotal) Entegrasyonu, Sysmon ile Derinlemesine Windows Log Analizi, SOAR (Slack Webhook) ve API Otomasyonu  
**Hazırlayan:** Kıvanç Bergama  
**Tarih:** 5 Ağustos 2026  

---

## 1. Giriş ve Proje Amacı

Bu çalışmanın amacı; Wazuh SIEM/XDR çözümü kullanılarak kurum içi tehdit avcılığı (threat hunting), Siber Tehdit İstihbaratı (CTI) entegrasyonu, gelişmiş uç nokta izleme (EDR/Sysmon) ve otomatik olay müdahale (SOAR/Webhook) mekanizmalarının uçtan uca uygulanabilirliğini doğrulamaktır. 

Proje kapsamında geleneksel Windows Olay Günlüklerinin (Event Logs) yetersiz kaldığı dosyasız (fileless) saldırılar ve bellek ihlalleri izlenmiş, zararlı dosya tespitinde VirusTotal API'si ile otomatik hash sorgulaması yapılmış ve elde edilen Level 12+ kritik alarmlar Slack kanalına anlık olarak aktarılmıştır.

---

## 2. Mimari ve Laboratuvar Ortamı

* **Wazuh SIEM Manager:** Ubuntu WSL2 / Docker Engine üzerinde koşmaktadır (`single-node-wazuh.manager-1`, Sürüm: `v4.8.1`).
* **Windows Uç Nokta (Agent 002):** Windows 11 (`win-agent`, IP: `172.31.192.1`), Microsoft Sysmon v15.x yüklü.
* **Linux Uç Nokta (Agent 001):** Ubuntu / WSL2 (`wsl-agent`, IP: `127.0.0.1`).
* **Harici Entegrasyonlar:** VirusTotal Public API v3, Slack Incoming Webhooks, Python 3 REST API İstemcisi.

---

## 3. Gerçekleştirilen Adımlar ve Yapılandırma

### 3.1. Microsoft Sysmon Entegrasyonu ve Derinlemesine İzleme
Geleneksel Windows Olay İzleyicisi'nin göremediği bellekten parola çalma (LSASS Dumping), PowerShell tabanlı zararlı çalıştırma ve process injection hareketlerini izlemek amacıyla uç noktaya Sysmon kurulmuştur.

1. **Sysmon Kurulumu ve Kural Dosyası:** Windows ajanı üzerine SwiftOnSecurity konfigürasyon şablonu kullanılarak Sysmon servisi tanımlanmıştır.
2. **Wazuh Agent Yapılandırması (`ossec.conf`):** Sysmon loglarının Wazuh Manager'a iletilmesi için Windows agent yapılandırmasına şu kanal eklenmiştir:

```xml
<localfile>
  <log_format>eventchannel</log_format>
  <location>Microsoft-Windows-Sysmon/Operational</location>
</localfile>
```

3. **Elde Edilen Kazanım:** Process Creation (Event ID 1), Network Connections (Event ID 3) ve ProcessAccess / LSASS erişim logları (Event ID 10) Wazuh Manager tarafından ayrıştırılabilir hale getirilmiştir.

### 3.2. File Integrity Monitoring (FIM) ve VirusTotal CTI Entegrasyonu
Sistemde oluşturulan veya indirilen şüpheli dosyaların otomatik olarak zararlı analizinden geçirilmesi sağlanmıştır.

1. **FIM Dizini Belirleme:** Windows ajanı üzerindeki indirme klasörü izlemeye alınmıştır:

```xml
<syscheck>
  <directories realtime="yes">C:\Users\KivancBergama\Downloads\testklasoru</directories>
</syscheck>
```

2. **VirusTotal API Yapılandırması (`ossec.conf` - Manager):** Manager tarafında ilgili API anahtarı eklenerek FIM modülü ile VirusTotal arasında otomatik tetikleyici tanımlanmıştır:

```xml
<integration>
  <name>virustotal</name>
  <api_key>YOUR_VIRUSTOTAL_API_KEY</api_key>
  <group>syscheck</group>
  <alert_format>json</alert_format>
</integration>
```

### 3.3. Webhook (Slack) Entegrasyonu ve SOAR Mekanizması
Üretilen yüksek seviyeli alarmların operasyon ekibine anında iletilmesi amacıyla Slack entegrasyonu yapılmıştır.

1. **Slack Webhook Tanımlaması (`ossec.conf`):** Seviyesi 12 ve üzeri olan tüm alarmları (VirusTotal pozitif eşleşmeleri dahil) kanala düşürecek entegrasyon ayarlanmıştır:

```xml
<integration>
  <name>slack</name>
  <hook_url>[https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK](https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK)</hook_url>
  <level>12</level>
  <alert_format>json</alert_format>
</integration>
```

### 3.4. Python REST API Otomasyonu
Sistemdeki tüm ajanların durumunu programatik olarak sorgulamak için `get_wazuh_agents.py` betiği geliştirilmiştir.

* **Kimlik Doğrulama:** `https://localhost:55000/security/user/authenticate` uç noktasına POST isteği atılarak JWT (JSON Web Token) alınmıştır.
* **Sorgulama:** Alınan Bearer token ile `https://localhost:55000/agents` uç noktasından aktif ajan verileri çekilmiştir.

<img width="1333" height="535" alt="Ekran görüntüsü 2026-08-05 104003" src="https://github.com/user-attachments/assets/f00e0924-7f15-4e47-ac35-d7b82ebe103e" />

*Şekil 3.1: `get_wazuh_agents.py` Python betiği ile Wazuh REST API üzerinden aktif ajan durumlarının ve IP verilerinin çekilmesi.*

---

## 4. Test, Doğrulama ve Saldırı Simülasyonu (EICAR)

Hazırlanan otomasyon zincirini doğrulamak amacıyla Windows uç noktasında (`win-agent`) izlenen dizin altında EICAR Antivirüs Test Dosyası (`eicar.txt`) oluşturulmuştur.

### Olay Akışı ve Log Analizi:
1. **Adım 1 (FIM Tespiti):** Dosya diske yazıldığı anda Wazuh FIM (syscheck) modülü değişikliği tespit etmiş ve varsayılan olarak **Rule ID 550 (Level 7)** seviyesinde yerel bütünlük değişikliği uyarısı vermiştir.
2. **Adım 2 (VirusTotal Sorgusu):** Wazuh Manager, FIM uyarısını yakalayarak dosya hash değerini arka planda VirusTotal API'sine göndermiştir.
3. **Adım 3 (Kritik Alarm Üretimi):** VirusTotal yanıtı döndüğünde 61 motorun dosyayı zararlı olarak işaretlediği görülmüş ve olay seviyesi **Rule ID 87105 (Level 12)** olarak güncellenmiştir.
4. **Adım 4 (Slack Bildirimi):** Level 12 kuralı tetiklendiği için Wazuh Slack entegrasyonu çalışmış ve saniyeler içinde ilgili kanala bildirim düşmüştür.

<img width="1780" height="307" alt="Ekran görüntüsü 2026-08-05 125239" src="https://github.com/user-attachments/assets/a8aa6064-4e27-4170-9c98-85e9278b9541" />

*Şekil 4.1: Wazuh Dashboard üzerindeki Güvenlik Alarmları (Security Alerts) tablosunda dosya değişikliği (Rule 550) ve hemen ardından tetiklenen VirusTotal zararlı tespiti (Rule 87105 - Level 12).*

<img width="770" height="357" alt="Ekran görüntüsü 2026-08-05 125216" src="https://github.com/user-attachments/assets/4acb2fd1-7e77-4452-b646-cade2fada9d6" />

*Şekil 4.2: Tetiklenen kritik seviyedeki (Level 12) zararlı tespiti uyarısının Slack Webhook entegrasyonu ile otomatik olarak kanala iletilmesi.*

---

## 5. Sonuç ve Değerlendirme

Bu çalışma ile birlikte;
* Standart OS loglarının ötesine geçilerek **Sysmon** vasıtasıyla süreç ve bellek düzeyinde görünürlük kazanılmıştır.
* Statik imza taraması yerine **VirusTotal CTI** entegrasyonu ile dinamik ve otomatize tehdit analizi sağlanmıştır.
* **Wazuh REST API** kullanılarak güvenlik operasyonlarının kod ile yönetilebilirliği (Infrastructure as Code / SOAR) kanıtlanmıştır.
* Uç noktada tespit edilen kritik bir zararlı yazılımın insan müdahalesine gerek kalmaksızın **Slack Webhook** üzerinden güvenlik ekibine anlık alarm olarak iletilmesi altyapısı başarıyla doğrulanmıştır.

---
# English
 
# CYBERSECURITY AND SIEM/XDR LABORATORY PROJECT REPORT

**Topic:** CTI (VirusTotal) Integration, Deep Windows Log Analysis with Sysmon, SOAR (Slack Webhook), and API Automation on Wazuh SIEM/XDR Platform  
**Prepared by:** Kıvanç Bergama  
**Date:** August 5, 2026  

---

## 1. Introduction and Project Objective

The objective of this study is to verify the end-to-end applicability of internal threat hunting, Cyber Threat Intelligence (CTI) integration, advanced endpoint monitoring (EDR/Sysmon), and automated incident response (SOAR/Webhook) mechanisms using the Wazuh SIEM/XDR solution. 

Within the scope of the project, fileless attacks and memory violations—where traditional Windows Event Logs fall short—were monitored, automated hash querying was performed with the VirusTotal API upon the detection of malicious files, and the resulting Level 12+ critical alerts were instantaneously forwarded to a Slack channel.

---

## 2. Architecture and Laboratory Environment

* **Wazuh SIEM Manager:** Running on Ubuntu WSL2 / Docker Engine (`single-node-wazuh.manager-1`, Version: `v4.8.1`).
* **Windows Endpoint (Agent 002):** Windows 11 (`win-agent`, IP: `172.31.192.1`), Microsoft Sysmon v15.x installed.
* **Linux Endpoint (Agent 001):** Ubuntu / WSL2 (`wsl-agent`, IP: `127.0.0.1`).
* **External Integrations:** VirusTotal Public API v3, Slack Incoming Webhooks, Python 3 REST API Client.

---

## 3. Executed Steps and Configuration

### 3.1. Microsoft Sysmon Integration and Deep Monitoring
Sysmon was installed on the endpoint to monitor memory credential theft (LSASS Dumping), PowerShell-based malicious execution, and process injection activities that the traditional Windows Event Viewer cannot detect.

1. **Sysmon Installation and Rule File:** The Sysmon service was configured on the Windows agent using the SwiftOnSecurity configuration template.
2. **Wazuh Agent Configuration (`ossec.conf`):** The following channel was added to the Windows agent configuration to forward Sysmon logs to the Wazuh Manager:

```xml
<localfile>
  <log_format>eventchannel</log_format>
  <location>Microsoft-Windows-Sysmon/Operational</location>
</localfile>
```

3. **Achieved Outcome:** Process Creation (Event ID 1), Network Connections (Event ID 3), and ProcessAccess / LSASS access logs (Event ID 10) were made parsable by the Wazuh Manager.

### 3.2. File Integrity Monitoring (FIM) and VirusTotal CTI Integration
Automated malicious analysis of suspicious files created or downloaded on the system was ensured.

1. **Determining the FIM Directory:** The download folder on the Windows agent was monitored:

```xml
<syscheck>
  <directories realtime="yes">C:\Users\KivancBergama\Downloads\testklasoru</directories>
</syscheck>
```

2. **VirusTotal API Configuration (`ossec.conf` - Manager):** An automated trigger between the FIM module and VirusTotal was defined by adding the relevant API key on the Manager side:

```xml
<integration>
  <name>virustotal</name>
  <api_key>YOUR_VIRUSTOTAL_API_KEY</api_key>
  <group>syscheck</group>
  <alert_format>json</alert_format>
</integration>
```

### 3.3. Webhook (Slack) Integration and SOAR Mechanism
Slack integration was established to instantly forward generated high-level alerts to the operations team.

1. **Slack Webhook Definition (`ossec.conf`):** An integration was configured to drop all alerts with a severity of 12 and above (including VirusTotal positive matches) into the channel:

```xml
<integration>
  <name>slack</name>
  <hook_url>[https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK](https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK)</hook_url>
  <level>12</level>
  <alert_format>json</alert_format>
</integration>
```

### 3.4. Python REST API Automation
The `get_wazuh_agents.py` script was developed to programmatically query the status of all agents in the system.

* **Authentication:** A JWT (JSON Web Token) was obtained by sending a POST request to the `https://localhost:55000/security/user/authenticate` endpoint.
* **Querying:** Active agent data was retrieved from the `https://localhost:55000/agents` endpoint using the acquired Bearer token.

<img width="1333" height="535" alt="Ekran görüntüsü 2026-08-05 104003" src="https://github.com/user-attachments/assets/68b625c1-ae6a-46df-ba3a-aeff5a7ae4d9" />

*Figure 3.1: Retrieving active agent statuses and IP data via the Wazuh REST API using the `get_wazuh_agents.py` Python script.*

---

## 4. Test, Validation, and Attack Simulation (EICAR)

To validate the prepared automation chain, the EICAR Antivirus Test File (`eicar.txt`) was created under the monitored directory on the Windows endpoint (`win-agent`).

### Event Flow and Log Analysis:
1. **Step 1 (FIM Detection):** As soon as the file was written to the disk, the Wazuh FIM (syscheck) module detected the change and issued a local integrity change warning at the default **Rule ID 550 (Level 7)** level.
2. **Step 2 (VirusTotal Query):** Catching the FIM alert, the Wazuh Manager sent the file hash value to the VirusTotal API in the background.
3. **Step 3 (Critical Alert Generation):** When the VirusTotal response returned, it was observed that 61 engines flagged the file as malicious, and the event level was updated to **Rule ID 87105 (Level 12)**.
4. **Step 4 (Slack Notification):** Because the Level 12 rule was triggered, the Wazuh Slack integration executed, and the notification dropped into the respective channel within seconds.

<img width="1780" height="307" alt="Ekran görüntüsü 2026-08-05 125239" src="https://github.com/user-attachments/assets/a2e87127-1cf1-43b1-991e-d6b00b9597e9" />

*Figure 4.1: File change (Rule 550) and the immediately triggered VirusTotal malicious detection (Rule 87105 - Level 12) in the Security Alerts table on the Wazuh Dashboard.*

<img width="770" height="357" alt="Ekran görüntüsü 2026-08-05 125216" src="https://github.com/user-attachments/assets/a8998749-d656-4abf-b66f-18ca14cc9461" />

*Figure 4.2: Automated delivery of the triggered critical level (Level 12) malicious detection alert to the channel via Slack Webhook integration.*

---

## 5. Conclusion and Evaluation

With this study;
* Visibility at the process and memory level was gained via **Sysmon**, going beyond standard OS logs.
* Dynamic and automated threat analysis was achieved with **VirusTotal CTI** integration instead of static signature scanning.
* The code-manageability of security operations (Infrastructure as Code / SOAR) was proven using the **Wazuh REST API**.
* The infrastructure to forward a critical malware detected at the endpoint as an instantaneous alert to the security team via **Slack Webhook**—without requiring human intervention—was successfully validated.
