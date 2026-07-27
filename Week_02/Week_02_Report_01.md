# Staj Günlük Raporu

**Tarih:** 27 Temmuz 2026
**Hazırlayan:** Kıvanç Bergama
**Çalışma Konusu:** Wazuh Üzerinde Dosya Bütünlüğü İzleme (FIM) Yapılandırması, MITRE ATT&CK ve Regülasyon Uyumluluk Analizi

Merkezi log yönetimi ve zafiyet yönetimi süreçleri kapsamında, sistemlerdeki yetkisiz değişiklikleri tespit edebilmek amacıyla Wazuh üzerinde Dosya Bütünlüğü İzleme (FIM) konfigürasyonunun yapılması hedeflenmiştir. Bu çalışma kapsamında; simülasyon testlerinin gerçekleştirilmesi ve elde edilen alarmların MITRE ATT&CK matrisi ile uluslararası mevzuat (PCI DSS, GDPR) panoları üzerinden detaylı bir şekilde analiz edilmesi amaçlanmıştır.

Bu hedefler doğrultusunda ilk olarak test ortamındaki (WSL) Wazuh ajanının `ossec.conf` yapılandırma dosyasına müdahale edilmiş ve `<syscheck>` modülü altındaki izleme listesine `/etc` dizini eklenmiştir. Varsayılan 12 saatlik tarama periyodu yerine, olayları gerçekleştiği an yakalayabilmek için `realtime="yes"` (gerçek zamanlı izleme) parametresi yapılandırmaya dahil edilmiştir. Ayrıca, modifiye edilen dosyaların içindeki metin değişimlerini (diff) loglara yansıtabilmek adına `report_changes="yes"` özelliği aktif edilmiş ve ajan servisi yeniden başlatılarak konfigürasyon devreye alınmıştır. Devamında FIM modülünün işlevselliğini test etmek amacıyla, terminal üzerinden `/etc` dizini içerisinde `fim_test.txt` isimli bir dosya oluşturulmuş, içerisine veri yazılarak manipüle edilmiş ve son olarak sistemden tamamen silinerek değişiklik simülasyonları tamamlanmıştır.

<img width="1918" height="833" alt="Ekran görüntüsü 2026-07-27 103642" src="https://github.com/user-attachments/assets/4b6190cb-56e2-458a-a2e1-4724e34c4a9f" />

Gerçekleştirilen simülasyonlar sonucu oluşan güvenlik logları, Wazuh Dashboard "Events" sekmesi üzerinden detaylıca incelenmiştir. İçerik analizi sonucunda, `report_changes` özelliğinin devreye girmesiyle birlikte `syscheck.diff` alanında dosyaya eklenen "Zararli bir icerik eklendi" metninin başarılı bir şekilde raporlandığı gözlemlenmiştir. İncelenen logların MITRE ATT&CK matrisiyle eşleşmesine bakıldığında, dosya silme eyleminin saldırganların izlerini kaybettirmek için kullandığı "Defense Evasion" (Savunmadan Kaçınma) taktiği ile ilişkilendirildiği saptanmıştır. Regülasyon uyumluluğu açısından ise, üretilen alarmların detaylarında gerçekleşen ihlallerin kredi kartı veri güvenliği standardı olan PCI DSS'in 11.5 numaralı maddesiyle ve Avrupa Birliği Kişisel Verileri Koruma Tüzüğü (GDPR) Madde II_5.1.f ile eşleştiği tespit edilmiştir.

Bu kapsamlı çalışma sonucunda FIM mekanizmasının, yalnızca teknik bir dosya değişim uyarısı üretmekle kalmadığı uygulamalı olarak tecrübe edilmiştir. Bu mekanizmanın aynı zamanda saldırganın amacını profilleyen ve kurumun uluslararası yasal standartlara (PCI DSS, KVKK/GDPR vb.) uyumluluğunu şeffaf bir şekilde denetçilere kanıtlayan stratejik bir güvenlik operasyon aracı olduğu kavranmıştır. Sonuç itibarıyla, kritik dizinler için yapılandırılan anlık izleme mekanizmasının ve log raporlama sisteminin başarıyla çalıştığı teyit edilmiştir.

---

# Internship Daily Report

**Date:** July 27, 2026
**Prepared by:** Kıvanç Bergama
**Subject:** File Integrity Monitoring (FIM) Configuration on Wazuh, MITRE ATT&CK, and Regulatory Compliance Analysis

Within the scope of centralized log management and vulnerability management processes, it was aimed to configure File Integrity Monitoring (FIM) on Wazuh to detect unauthorized changes in the systems. In this context, the objective was to perform simulation tests and analyze the resulting alerts through the MITRE ATT&CK matrix and international regulatory (PCI DSS, GDPR) compliance dashboards.

In line with these objectives, the `ossec.conf` configuration file of the Wazuh agent in the test environment (WSL) was first modified, and the `/etc` directory was added to the monitoring list under the `<syscheck>` module. Instead of the default 12-hour scanning period, the `realtime="yes"` parameter was included in the configuration to capture events as they occur. Additionally, to reflect the text changes (diff) inside the modified files in the logs, the `report_changes="yes"` feature was enabled, and the agent service was restarted to activate the configuration. Subsequently, to test the functionality of the FIM module, a file named `fim_test.txt` was created within the `/etc` directory via the terminal, manipulated by writing data into it, and finally completely deleted from the system, thereby completing the change simulations.

<img width="1918" height="833" alt="Ekran görüntüsü 2026-07-27 103642" src="https://github.com/user-attachments/assets/4a2a7da8-904f-45d7-bb27-7f09783668f4" />

The security logs generated as a result of the simulations were examined in detail via the Wazuh Dashboard "Events" tab. As a result of the content analysis, with the activation of the `report_changes` feature, it was observed that the text "Zararli bir icerik eklendi" added to the file was successfully reported in the `syscheck.diff` field. When examining the alignment of the analyzed logs with the MITRE ATT&CK matrix, it was determined that the file deletion action was associated with the "Defense Evasion" tactic used by attackers to cover their tracks. Regarding regulatory compliance, it was identified in the alert details that the occurring violations matched Requirement 11.5 of the Payment Card Industry Data Security Standard (PCI DSS) and Article II_5.1.f of the European Union General Data Protection Regulation (GDPR).

As a result of this comprehensive study, it was practically experienced that the FIM mechanism does not solely generate a technical file change alert. It was understood that this mechanism is simultaneously a strategic security operations tool that profiles the attacker's intent and transparently proves the organization's compliance with international legal standards (PCI DSS, KVKK/GDPR, etc.) to auditors. In conclusion, it was confirmed that the real-time monitoring mechanism and log reporting system configured for critical directories are operating successfully.
