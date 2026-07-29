# Wazuh Özel Kural (Custom Rule) Geliştirme ve Test Raporu 
Tarih: 29.07.2026
*(Scroll down for the English version)*

## Araştırma ve Dekoder Analizi
Çalışmanın ilk aşamasında, sisteme varsayılan olarak gelen `/var/ossec/ruleset/rules/0095-sshd_rules.xml` dosyası incelenerek SSH başarılı giriş kuralı olan "Rule ID: 5715" referans (parent rule) olarak belirlenmiştir. Yapılan analizde Wazuh'un standart `sshd` dekoderinin, hedef kullanıcı (dstuser) ve kaynak IP (srcip) gibi verileri halihazırda sorunsuz bir şekilde ayrıştırdığı (parse ettiği) görülmüş; bu nedenle sisteme ekstra bir dekoder yazılmasına gerek kalmamıştır.

## Kuralın Geliştirilmesi (Custom Rule Creation)
Kuralın geliştirilmesi sürecinde, güncellemelerden etkilenmemesi adına yeni kural, güvenli alan olarak belirlenen `/var/ossec/etc/rules/local_rules.xml` dizinine eklenmiştir. Wazuh Manager (Docker) ortamında herhangi bir metin editörü (nano/vim) bulunmadığı için dosya yapılandırması "Here-Doc" (`cat << EOF`) yöntemiyle dışarıdan içeri aktarılarak (inject) çözülmüştür. Oluşturulan 100001 ID'li kural, `if_sid` parametresi kullanılarak 5715 numaralı ana kurala bağlanmıştır. Ayrıca `time` parametresi, canlı test yapılabilmesi için UTC saat dilimi göz önünde bulundurularak 07:00-12:00 aralığına ayarlanmış ve `user` parametresiyle "root", "admin" veya "administrator" girişlerini yakalayacak şekilde özel olarak yapılandırılmıştır.

**Oluşturulan Kuralın Terminal Çıktısı:**
<img width="1154" height="260" alt="Ekran görüntüsü 2026-07-29 130540" src="https://github.com/user-attachments/assets/2613286d-fb86-4107-a44a-59a4dbed9db3" />

## Test ve Canlı Simülasyon Aşaması
Test ve canlı simülasyon aşamasında karşılaşılan log iletimi sorunu (Troubleshooting 1), test ortamının WSL (Windows Subsystem for Linux) olması sebebiyle varsayılan `rsyslog` servisinin kapalı olmasından kaynaklanmıştır. Bu sebeple standart `logger` komutu yerine, log doğrudan ajanın (Agent) okuduğu `/var/log/auth.log` dosyasına manuel olarak yazdırılarak sorun çözülmüştür. İkinci bir sorun olan tarih filtresini (Troubleshooting 2) aşmak ve Wazuh Dashboard'un bu logları yakalayabilmesini sağlamak için, log içerisine `$(date +"%b %d %H:%M:%S")` komutuyla güncel saat damgası (timestamp) eklenmiştir. Bu adımların sonucunda, kuralın ilgili zaman dilimi içerisinde başarıyla tetiklendiği ve Wazuh Dashboard üzerindeki Security Events ekranına Level 12 alarmı olarak düştüğü doğrulanmıştır.

**Wazuh Dashboard Üzerinde Tetiklenen Alarm:**
<img width="1808" height="707" alt="Ekran görüntüsü 2026-07-29 132310" src="https://github.com/user-attachments/assets/05d2487e-58b3-46fe-b78a-fa2081828207" />

## Sistemin Temizlenmesi (Cleanup Phase)
Son olarak sistemin temizlenmesi aşamasında, görev başarıyla tamamlandıktan sonra ilerleyen süreçlerde log kirliliği (False Positive) yaratmaması için kural pasifize edilmek istenmiştir. Bu süreçte karşılaşılan Wazuh XML Parser sorununda (Troubleshooting 3), kural silinip dosya tamamen boşaltıldığında Wazuh'un katı yapısı ("Error 1202" ve "Group without any rule" hataları) nedeniyle servisin yeniden başlamadığı görülmüştür. Çözüm olarak; sistemi ve XML yapısını bozmamak adına, Level değeri "0" (Sıfır) olan ve loglar içinde "AslaGelmeyecekBirKelime" arayan zararsız bir hayalet kural (Dummy Rule) yazılarak servis hatasız bir şekilde yeniden başlatılmış ve sistem güvenli haline geri döndürülmüştür.

---

# Wazuh Custom Rule Development and Testing Report
Date: 29.07.2026
## Research and Decoder Analysis
In the first phase of the study, the default `/var/ossec/ruleset/rules/0095-sshd_rules.xml` file was analyzed, and the successful SSH login rule, "Rule ID: 5715", was designated as the reference (parent rule). The analysis revealed that Wazuh's standard `sshd` decoder already successfully parsed data such as the target user (dstuser) and source IP (srcip). Therefore, there was no need to write an additional decoder for the system.

## Custom Rule Creation
During the rule development process, the new rule was added to the `/var/ossec/etc/rules/local_rules.xml` directory, which is defined as a safe zone, to ensure it would not be overwritten by system updates. Since there is no built-in text editor (nano/vim) in the Wazuh Manager (Docker) environment, the file configuration was handled by injecting it externally using the "Here-Doc" (`cat << EOF`) method. The created rule, with ID 100001, was linked to the main rule 5715 using the `if_sid` parameter. Furthermore, the `time` parameter was set to the 07:00-12:00 range, taking the UTC time zone into account for live testing, and the `user` parameter was specifically configured to capture "root", "admin", or "administrator" login attempts.

**Terminal Output of the Created Rule:**
<img width="1154" height="260" alt="Ekran görüntüsü 2026-07-29 130540" src="https://github.com/user-attachments/assets/f9bdfdbc-bd5f-47fd-9ae9-67aefc120c1c" />

## Testing and Live Simulation Phase
The log transmission issue (Troubleshooting 1) encountered during the testing and live simulation phase originated from the default `rsyslog` service being inactive because the test environment was WSL (Windows Subsystem for Linux). To resolve this, instead of using the standard `logger` command, the logs were manually appended directly into the `/var/log/auth.log` file, which the Agent monitors. To bypass the second issue, a date filtering mechanism (Troubleshooting 2), and to ensure the Wazuh Dashboard could capture these logs, a current timestamp was injected into the log structure using the `$(date +"%b %d %H:%M:%S")` command. As a result of these steps, it was confirmed that the rule was successfully triggered within the specified time frame and appeared as a Level 12 alert on the Security Events screen of the Wazuh Dashboard.

**Triggered Alert on Wazuh Dashboard:**
<img width="1808" height="707" alt="Ekran görüntüsü 2026-07-29 132310" src="https://github.com/user-attachments/assets/7f2f9d19-dfb7-428e-b4db-959133605e91" />

## System Cleanup (Cleanup Phase)
Finally, during the cleanup phase, after the task was successfully accomplished, it was necessary to pacify the rule to prevent it from causing log pollution (False Positives) in future operations. During this phase, a Wazuh XML Parser issue (Troubleshooting 3) was encountered; when the rule was deleted and the file was left completely empty, the service failed to restart due to Wazuh's strict XML enforcement ("Error 1202" and "Group without any rule" errors). As a solution, to avoid corrupting the system and the XML structure, a harmless "Dummy Rule" with a Level value of "0" (Zero)—configured to match an impossible string like "AslaGelmeyecekBirKelime" (A Word That Will Never Come)—was implemented. The service was successfully restarted without any errors, and the system was safely restored to its initial state.
