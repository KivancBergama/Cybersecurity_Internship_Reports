# Gün 1: Wazuh ile Yerel SOC Laboratuvarı Kurulumu ve Tehdit Analizi

## Teorik Altyapı ve Hazırlık
Bugünkü staj çalışmalarımda öncelikli olarak SIEM ve XDR kavramlarının teorik altyapısını öğrenerek işe başladım. Bu kapsamda Wazuh mimarisini (Manager, Indexer, Dashboard, Agent) detaylıca inceledim; ayrıca Linux (Ubuntu Server) temel komutları ile Docker mimarisinin mantığını gözden geçirdim. Teorik araştırmalarımın ardından, Wazuh sunucu ortamını ayağa kaldırma ve ilk logları toplama görevine geçerek yerel bir SOC (Güvenlik Operasyon Merkezi) laboratuvar ortamı kurdum.

## Laboratuvar Kurulumu ve Uç Nokta Entegrasyonu
Bu laboratuvar ortamında merkezi log yönetimi ve SIEM entegrasyonu üzerine uygulamalı testler gerçekleştirdim. Çalışmalarıma, yapılandırdığım Wazuh merkezi yönetim paneline (Dashboard) ilk uç nokta cihazını entegre ederek başladım. Hedef makine olarak Windows Subsystem for Linux (WSL) altyapısında çalışan Ubuntu dağıtımını belirledim. Arayüz üzerinden Linux/amd64 mimarisine uygun Debian (.deb) kurulum paketini komut satırı aracılığıyla indirerek sisteme entegre ettim. Yükleme işleminin ardından `systemctl` yöneticisi ile Wazuh Agent servisini arka planda aktif hale getirerek, uç noktanın Wazuh Manager ile başarılı bir şekilde haberleşmesini sağladım ve ajanın "Active" statüsüne geçtiğini doğruladım.

## Tehdit Simülasyonu: SSH Brute-Force
Merkezi log toplama mekanizmasının tespit yeteneklerini test edebilmek amacıyla, izole WSL ortamında bir yetkisiz erişim (unauthorized access) simülasyonu kurguladım. Minimal WSL yapısında varsayılan olarak kapalı gelen SSH servislerini aktifleştirmek için `openssh-server` paketinin kurulumunu gerçekleştirdim. Gerekli RSA, ECDSA ve ED25519 güvenlik anahtarlarının oluşturulmasının ardından servisi ayağa kaldırarak hedef sistemi log üretimine hazır hale getirdim.

Oluşturduğum altyapı üzerinden SSH servisine yönelik kontrollü bir kaba kuvvet (brute-force) saldırısı simüle ettim. Sistemde var olmayan sahte bir kullanıcı adıyla (`testkullanicisi@localhost`) yerel ağ üzerinden art arda bağlantı istekleri gönderdim. Bilinçli olarak defalarca yanlış parola girerek hedef sistemin "Permission denied" yanıtı vermesini tetikledim. Bu işlem sayesinde, Linux sistemlerinin kimlik doğrulama mekanizmalarında standart olarak oluşan "Authentication failure" olay günlüklerinin (log) başarılı bir şekilde üretilmesini sağladım.

<img width="1018" height="203" alt="Ekran görüntüsü 2026-07-20 162418" src="https://github.com/user-attachments/assets/a29cdcfe-0bfa-437c-83a2-3056cf8d6997" />

## Tehdit Avcılığı (Threat Hunting) ve Analiz
Son aşamada, ürettiğim güvenlik ihlali loglarının SIEM tarafındaki yansımalarını incelemek üzere Wazuh Dashboard'un Threat Hunting (Tehdit Avı) modülünü kullandım. Gerçek zamanlı olarak panele düşen alarmları analiz ederek; başarısız giriş denemelerinin PAM ve SSHD logları üzerinden nasıl yakalandığını gözlemledim. 

<img width="1790" height="762" alt="Ekran görüntüsü 2026-07-20 143813" src="https://github.com/user-attachments/assets/2736e1e0-e8ad-4016-9904-cb6b6e79ca4d" />

Tespit edilen olayların MITRE ATT&CK matrisinde "Credential Access" taktiği ve "T1110" (Brute Force) tekniği ile otomatik olarak eşleştirildiğini analiz ettim. 

<img width="1897" height="393" alt="Ekran görüntüsü 2026-07-20 143825" src="https://github.com/user-attachments/assets/42b81a46-d937-4e6f-bbc3-8aded59ed313" />

Ayrıca bu güvenlik ihlallerinin, PCI DSS (10.2.4 ve 10.2.5) gibi uluslararası uyumluluk standartlarına göre oluşturduğu riskleri grafiksel veriler üzerinden raporlayarak bugünkü hedeflerimi başarıyla tamamladım.

<img width="1800" height="826" alt="Ekran görüntüsü 2026-07-20 143838" src="https://github.com/user-attachments/assets/f95929e7-122c-4394-8916-797082f281ca" />

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
English

# Day 1: Local SOC Laboratory Setup and Threat Analysis with Wazuh

## Theoretical Foundation and Preparation
During today's internship, I initially focused on the theoretical foundations of SIEM and XDR concepts. In this context, I examined the Wazuh architecture (Manager, Indexer, Dashboard, Agent) in detail, and reviewed basic Linux (Ubuntu Server) commands alongside the underlying logic of Docker containerization. Following this theoretical research, I deployed the Wazuh server environment and established a local SOC (Security Operations Center) laboratory to begin collecting endpoint logs.

## Laboratory Setup and Endpoint Integration
Within this laboratory environment, I conducted practical tests focusing on centralized log management and SIEM integration. I started by integrating the first endpoint device into the configured Wazuh central management panel (Dashboard). I selected an Ubuntu distribution running on the Windows Subsystem for Linux (WSL) as the target machine. Through the command line interface, I downloaded and installed the appropriate Debian (.deb) package for the Linux/amd64 architecture. Following the installation, I activated the Wazuh Agent service in the background using the `systemctl` manager, ensuring successful communication between the endpoint and the Wazuh Manager, and verified that the agent's status transitioned to "Active".

## Threat Simulation: SSH Brute-Force
To test the detection capabilities of the centralized log collection mechanism, I configured an unauthorized access simulation within the isolated WSL environment. I installed the `openssh-server` package to enable SSH services, which are disabled by default in the minimal WSL structure. After generating the necessary RSA, ECDSA, and ED25519 security keys, I initiated the service, making the target system ready for log generation.

Using the established infrastructure, I simulated a controlled brute-force attack against the SSH service. I repeatedly sent connection requests over the local network using a non-existent, fake username (`testkullanicisi@localhost`). By intentionally entering incorrect passwords multiple times, I triggered the target system to return "Permission denied" responses. Through this process, I successfully generated standard "Authentication failure" event logs, which are intrinsic to Linux authentication mechanisms.

<img width="1018" height="203" alt="Ekran görüntüsü 2026-07-20 162418" src="https://github.com/user-attachments/assets/11f53f14-7473-4be5-8e77-dd78bc7ca02a" />

## Threat Hunting and Analysis
In the final phase, I utilized the Threat Hunting module of the Wazuh Dashboard to analyze how the generated security breach logs were reflected on the SIEM side. By monitoring real-time alerts on the dashboard, I observed how failed login attempts were accurately captured via PAM and SSHD logs. 

<img width="1790" height="762" alt="Ekran görüntüsü 2026-07-20 143813" src="https://github.com/user-attachments/assets/82a06a30-3ddb-4a4f-9563-d7e313d2e4cf" />

I verified that the detected events were automatically mapped to the "Credential Access" tactic and the "T1110" (Brute Force) technique within the MITRE ATT&CK framework. 

<img width="1897" height="393" alt="Ekran görüntüsü 2026-07-20 143825" src="https://github.com/user-attachments/assets/704b00c9-232b-4404-8942-0e1326243107" />

Furthermore, I successfully concluded today's objectives by visually reporting the risks these security breaches pose in relation to international compliance standards, specifically PCI DSS (Requirements 10.2.4 and 10.2.5).

<img width="1800" height="826" alt="Ekran görüntüsü 2026-07-20 143838" src="https://github.com/user-attachments/assets/f061900a-d090-499d-a10c-aaad7ad35424" />

