# Modern Tehdit Simülasyonu - "Benign Malware"

Bu proje, siber güvenlik analistlerinin (Blue Team), forensik uzmanlarının ve öğrencilerin **zararlı yazılım davranışlarını** güvenli bir ortamda analiz etmelerini sağlamak amacıyla Go dili ile geliştirilmiştir.

Gerçek bir zararlı yazılımın *davranışlarını* (behavior) taklit eder ancak sisteminize kalıcı zarar vermez, verilerinizi şifrelemez veya çalmaz.

## ⚠️ Güvenlik ve Uyarılar

Bu yazılım **yalnızca eğitim amaçlıdır**. Antivirüs yazılımları ve EDR çözümleri tarafından "zararlı" olarak algılanması normaldir ve tasarımın bir parçasıdır.

**Güvenlik Mekanizması (Kill Switch):**
Yazılımın yanlışlıkla gerçek sistemlerde çalışmasını önlemek için bir çevre değişkeni kontrolü eklenmiştir.
`ENV_TRAINING_MODE=TRUE` değişkeni ayarlanmadığı sürece yazılım **ÇALIŞMAZ**.

## 🛠️ Modüler Mimari

Yazılım, `main.go` orkestratörü tarafından yönetilen ve eşzamanlı (concurrent) çalışan 3 ana modülden oluşur:

### 1. Gözetim Modülü (`internal/surveillance`)
Bir casus yazılımın (Spyware/Infostealer) bilgi toplama aktivitelerini simüle eder.
*   **Keylogger (Tuş Kaydedici):** `GetAsyncKeyState` API'sini kullanarak tuş vuruşlarını dinler. (Sadece API çağrısı üretir, veriyi kaydetmez).
*   **Clipboard (Pano) Monitörü:** Panoya erişmek için `OpenClipboard` API'sini düzenli olarak çağırır.
*   **Browser Launcher:** `rundll32.exe` kullanarak gizli bir şekilde tarayıcı üzerinden bağlantı açmaya çalışır (Living off the Land).

### 2. Kaynak Modülü (`internal/resource`)
Sistemi yoran veya bellekte saklanan zararlıların (Miner, Ransomware) izlerini simüle eder.
*   **Memory Bloater (Bellek Şişirme):** Bellekte 50MB'lık bir alan tahsis eder ve içine `AES-256-KEY` ve `DEADBEEF` gibi sahte "imzalar" (flags) yerleştirir. Bu, **Volatility** gibi bellek analizi araçlarıyla tespit edilmeyi bekleyen bir artefaktır.
*   **CPU Spiker (İşlemci Yükü):** "Testere Dişi" (Sawtooth) modeliyle çalışır. 2 saniye yoğun işlem (mining simülasyonu), 5 saniye uyku. Bu, **Process Hacker** grafiklerinde belirgin bir anomali yaratır.

### 3. Ağ Modülü (`internal/network`)
Komuta Kontrol (C2) sunucusu ile haberleşen bir Botnet ajanını simüle eder.
*   **Beaconing:** Düzenli aralıklarla yerel bir adrese (`127.0.0.1:8080`) HTTP POST istekleri gönderir.
*   **Jitter (Rastgelelik):** Düzenli aralıkları (30sn) bozmak ve IDS/Firewall tespiti zorlaştırmak için %20 oranında rastgele zaman sapması (jitter) uygular.
*   **İmzalar:** `X-Session-ID` gibi şüpheli HTTP başlıkları ve manipüle edilmiş `User-Agent` bilgisi gönderir.

## 🕵️‍♂️ Forensik Analiz İpuçları

Bu yazılımı analiz ederken şunlara dikkat edin:

1.  **Process Hacker / Task Manager:**
    *   CPU kullanımındaki ritmik artışları gözlemleyin.
    *   `rundll32.exe` gibi alt süreçleri (child processes) inceleyin.
    *   Sürecin bellek kullanımının (Private Bytes) yapay olarak yüksek olduğunu fark edin.

2.  **Volatility (Bellek Forensiği):**
    *   `windows.yarascan` komutu ile `DEADBEEF` veya `MALWARE_SKELETON_KEY` dizelerini arayın.
    *   `windows.pslist` ile süreci tespit edin.

3.  **Wireshark / Ağ Analizi:**
    *   HTTP trafiğindeki `X-Session-ID` başlığını yakalayın.
    *   Paketler arasındaki zaman farklarını (Delta Time) inceleyerek makine kaynaklı bir trafik olduğunu teyit edin.

## 🚀 Kurulum ve Çalıştırma

### Gereksinimler
*   Go 1.18+ (Kodu derlemek için)
*   Windows İşletim Sistemi (Hedef platform)

### Derleme (Build)
```powershell
go build -o benign_malware.exe .
```

### Çalıştırma (Güvenli Modda)
PowerShell üzerinden çalıştırmak için:
```powershell
$env:ENV_TRAINING_MODE='TRUE'; .\benign_malware.exe
```

Simülasyon 30 dakika sonra otomatik olarak kapanacaktır (Time-to-Live). Manuel durdurmak için `CTRL+C` yapabilirsiniz.
"# Zararl-Yaz-l-m-Analizi" 
