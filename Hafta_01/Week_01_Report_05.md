# Faaliyet Raporu: SIEM Ortamında Zafiyet Tespiti (Vulnerability Detection) ve Analizi

**Hazırlayan:** Kıvanç Bergama
**Tarih:** 24 Temmuz 2026
**Uygulama Ortamı:** Docker (Wazuh Manager) ve WSL Ubuntu 24.04 LTS (Wazuh Agent)

---

## 1. Çalışmanın Amacı
Bu laboratuvar çalışmasında, Wazuh SIEM çözümünün uç nokta (endpoint) üzerindeki zafiyetleri tespit etme yeteneklerini doğrulamak hedeflenmiştir. Bu kapsamda, Docker üzerinde koşan Wazuh Manager ve WSL üzerinde çalışan Ubuntu 24.04 LTS ajanı (agent) kullanılarak izole bir test ortamı hazırlanmıştır. Çalışmanın temel amacı, hedef sisteme bilerek zafiyet barındıran eski sürüm bir yazılım kurarak, Wazuh'un bu CVE (Common Vulnerabilities and Exposures) kayıtlarını yakalama ve raporlama sürecini uygulamalı olarak test etmektir.

## 2. Test Ortamının Hazırlanması ve Tarama Süreci
İşleme başlamadan önce, Wazuh Manager tarafındaki log kayıtları incelenerek zafiyet tarama (**vulnerability-detection**) modülünün ve işletim sistemlerine ait CVE veri tabanı akışlarının aktif olarak çalıştığı teyit edilmiştir. Ardından, güncel paket yöneticilerinin güvenlik engellerine takılmamak ve sistemi gerçek bir riske atmadan yalnızca envanter kaydı oluşturmak amacıyla yamalanmamış eski bir Apache sürümü (`apache2-bin_2.4.52-1ubuntu4_amd64.deb`) sisteme manuel olarak indirilmiştir. 

Bu paket, bağımlılıkları göz ardı eden zorunlu kurulum parametresiyle (`dpkg -i --force-depends`) sisteme tanıtılmış, böylece arka planda aktif bir web sunucusu çalıştırılmadan uygulamanın sistem envanterine girmesi sağlanmıştır. Envanter güncellemelerinin Manager'a anında iletilmesi için ajan servisi yeniden başlatılarak tarama süreci tetiklenmiştir.

## 3. Tespit ve Raporlama
Wazuh Dashboard üzerinden yapılan izlemeler sonucunda, ajanın **syscollector** modülü tarafından toplanan yeni envanterin Manager tarafından başarıyla analiz edildiği görülmüştür. Kurulan eski Apache sürümüne ait kritik ve yüksek seviyeli açıklar (örneğin **CVE-2024-27316**) ajan makinesinin "Vulnerabilities" paneline düşmüş ve bu tespitler filtrelenerek raporlama amacıyla ekran görüntüleriyle kayıt altına alınmıştır.

<img width="1792" height="911" alt="Ekran görüntüsü 2026-07-24 123251" src="https://github.com/user-attachments/assets/8571f670-94df-4992-b841-b6efcf25e050" />


## 4. Sistem Sıkılaştırma ve Temizlik (Post-Lab Cleanup)
Test sürecinin başarıyla tamamlanmasının ardından, siber güvenlik laboratuvar pratikleri gereği sistem temizliği aşamasına geçilmiştir. Zafiyet barındıran paket sistemden tamamen kaldırılarak paket yöneticisinde oluşan bağımlılık hataları onarılmış ve ajan servisi son kez yeniden başlatılarak test ortamının ilk halindeki güvenli durumuna dönmesi sağlanmıştır. 

**Sonuç:** Bu çalışmayla birlikte Wazuh'un OVAL veri tabanlarını kullanarak işletim sistemi seviyesindeki zafiyetleri tespit etme mekanizması ve sistem temizliğinin önemi pratik olarak kavranmıştır.

---

# English Version

# Activity Report: Vulnerability Detection and Analysis in SIEM Environment

**Prepared by:** Kıvanç Bergama
**Date:** July 24, 2026
**Application Environment:** Docker (Wazuh Manager) and WSL Ubuntu 24.04 LTS (Wazuh Agent)

---

## 1. Objective of the Study
In this laboratory study, the objective was to verify the vulnerability detection capabilities of the Wazuh SIEM solution on an endpoint. In this context, an isolated test environment was prepared using the Wazuh Manager running on Docker and an Ubuntu 24.04 LTS agent running on WSL. The main purpose of the study is to practically test the process of Wazuh capturing and reporting CVE (Common Vulnerabilities and Exposures) records by intentionally installing an older software version containing vulnerabilities on the target system.

## 2. Preparation of the Test Environment and Scanning Process
Before starting the process, the log records on the Wazuh Manager side were examined to confirm that the vulnerability scanning (**vulnerability-detection**) module and the CVE database feeds of the operating systems were actively working. Then, to bypass the security barriers of current package managers and solely create an inventory record without exposing the system to actual risk, an unpatched old version of Apache (`apache2-bin_2.4.52-1ubuntu4_amd64.deb`) was manually downloaded to the system. 

This package was introduced to the system using a forced installation parameter (`dpkg -i --force-depends`) that ignores dependencies, thus ensuring the application entered the system inventory without running an active web server in the background. To instantly transmit inventory updates to the Manager, the agent service was restarted, triggering the scanning process.

## 3. Detection and Reporting
As a result of the monitoring performed via the Wazuh Dashboard, it was observed that the new inventory collected by the agent's **syscollector** module was successfully analyzed by the Manager. Critical and high-severity vulnerabilities belonging to the installed old Apache version (e.g., **CVE-2024-27316**) appeared in the "Vulnerabilities" panel of the agent machine. These detections were filtered and recorded with screenshots for reporting purposes.

<img width="1792" height="911" alt="Ekran görüntüsü 2026-07-24 123251" src="https://github.com/user-attachments/assets/75f88b0d-0862-418a-8bbe-bdaa3a82d5bc" />

## 4. System Hardening and Cleanup (Post-Lab Cleanup)
Following the successful completion of the testing process, the system cleanup phase was initiated in accordance with cybersecurity laboratory best practices. The vulnerable package was completely removed from the system, the dependency errors in the package manager were repaired, and the agent service was restarted one last time to return the test environment to its initial secure state. 

**Conclusion:** Through this study, the mechanism of Wazuh detecting operating system-level vulnerabilities using OVAL databases and the importance of system cleanup were practically understood.
