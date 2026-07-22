# Wazuh XDR/SIEM Güvenlik Simülasyon Raporu / Security Simulation Report

---

## 🇹🇷 Türkçe (Turkish)

### 1. Tehdit Algılama (Detection) ve Log İzleme
Çalışmalar kapsamında öncelikle tehdit algılama (detection) yapılandırması ele alınmış olup, **Wazuh XDR/SIEM** platformu üzerinde uç nokta (endpoint) log izleme mimarisi test edilmiştir. 

Bu doğrultuda:
* **WSL (Ubuntu)** ajanı üzerinden alınan kimlik doğrulama loglarının (`auth.log`) merkezi Manager'a iletimi ve analizi sağlanmıştır. 
* Sistem üzerindeki güvenlik kurallarının test edilmesi amacıyla, Ubuntu ortamında **Hydra** aracı kullanılarak SSH servisine yönelik kontrollü bir Kaba Kuvvet (Brute-Force) saldırı simülasyonu gerçekleştirilmiştir.

### 2. Tespit ve Alarmlar
Yapılan simülasyon sonucunda, sistemin başarısız giriş denemelerini başarılı bir şekilde tespit ettiği gözlemlenmiştir. Aşağıdaki Wazuh kontrol panelinde (dashboard) saldırı anındaki log artışları ve kimlik doğrulama hatalarındaki (Authentication failure) sıçrama net bir şekilde görülmektedir:

<img width="1886" height="972" alt="Ekran görüntüsü 2026-07-22 145403" src="https://github.com/user-attachments/assets/ede6cb19-136e-4208-877d-b0cbcbc8951b" />

Saldırının yoğunlaşmasıyla birlikte sistemin eşik değerleri (threshold) aşılmış ve Wazuh Dashboard üzerinde aşağıdaki kritik güvenlik alarmları yakalanmıştır:

* **Kural 5551:** `PAM: Multiple failed logins in a small period of time` (Seviye 10)
* **Kural 5763:** `sshd: brute force trying to get access to the system` (Seviye 10)

Aşağıdaki görselde, tetiklenen Seviye 5 ve Seviye 10 alarmların zaman damgalı log detayları yer almaktadır:

<img width="1535" height="922" alt="Ekran görüntüsü 2026-07-22 150029" src="https://github.com/user-attachments/assets/2ce1ffcd-42ee-4dd5-99ab-1039b1a6a900" />

Elde edilen JSON formatındaki ham (raw) log çıktıları **"Sample Alert"** (Örnek Alarm) olarak raporlanmak üzere kayıt altına alınmıştır.

### 3. Otonom Savunma (Active Response) ve Beyaz Liste (Whitelist)
Son aşamada ise sistemin otonom savunma yetenekleri ve **Beyaz Liste (Whitelist)** mekanizması incelenmiştir. Tespit edilen saldırılara karşı sistemin otomatik aksiyon alabilmesi için çeşitli yapılandırmalar test edilmiştir:

* `ossec.conf` konfigürasyon dosyasına müdahale edilmiş ve `<active-response>` modülü içerisine `firewall-drop` kuralları eklenmiştir. 
* Localhost (`127.0.0.1`) üzerinden yapılan simülasyonlarda, engelleme kuralı tetiklenmesine rağmen XDR sisteminin otonom koruma refleksleri gözlemlenmiştir. 
* Sistemin kendi iç iletişimini ve çekirdek (kernel) kararlılığını korumak amacıyla yerel IP'yi "Beyaz Liste"de tuttuğu, bu sayede ağ kesintisini önleyerek IP bloklama işlemini kasti olarak pas geçtiği detaylı olarak analiz edilmiştir.

---

## 🇬🇧 İngilizce (English)

### 1. Threat Detection and Log Monitoring
Within the scope of the studies, the threat detection configuration was primarily addressed, and the endpoint log monitoring architecture was tested on the **Wazuh XDR/SIEM** platform.

In this context:
* The transmission and analysis of authentication logs (`auth.log`) obtained via the **WSL (Ubuntu)** agent to the central Manager was ensured.
* In order to test the security rules on the system, a controlled Brute-Force attack simulation targeting the SSH service was performed using the **Hydra** tool in the Ubuntu environment.

### 2. Detection and Alerts
As a result of the simulation, it was observed that the system successfully detected failed login attempts. The Wazuh dashboard below clearly shows the spike in log events and authentication failures during the attack:

<img width="1886" height="972" alt="Ekran görüntüsü 2026-07-22 145403" src="https://github.com/user-attachments/assets/8d4cad07-5b42-4db1-b0dc-a52025d7b91f" />

As the attack intensified, the system's threshold values were exceeded, and the following critical security alerts were captured on the Wazuh Dashboard:

* **Rule 5551:** `PAM: Multiple failed logins in a small period of time` (Level 10)
* **Rule 5763:** `sshd: brute force trying to get access to the system` (Level 10)

The following image shows the timestamped log details of the triggered Level 5 and Level 10 alerts:

<img width="1535" height="922" alt="Ekran görüntüsü 2026-07-22 150029" src="https://github.com/user-attachments/assets/abfb77db-0b5f-49c5-bee0-b992b4ea03fd" />

The obtained raw log outputs in JSON format were recorded to be reported as a **"Sample Alert"**.

### 3. Autonomous Defense (Active Response) and Whitelisting
In the final stage, the system's autonomous defense (active response) capabilities and **Whitelist** mechanism were examined. To enable the system to take automatic action against detected attacks, various configurations were tested:

* The `ossec.conf` configuration file was modified, and `firewall-drop` rules were added to the `<active-response>` module.
* In simulations conducted over Localhost (`127.0.0.1`), autonomous protection reflexes of the XDR system were observed despite the blocking rule being triggered.
* It was analyzed in detail that the system kept the local IP in the "Whitelist" to protect its internal communication and kernel stability, thus intentionally bypassing the IP blocking process to prevent network disruption.
