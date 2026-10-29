# Bölüm 1 — Raspberry Pi 5 Kurulumu (Ekransız / Headless)

> **Kaynak:** Marcelo Rovai, *Edge AI Engineering — Hands-On with the Raspberry Pi*, s. 26–80
> **Donanım:** Raspberry Pi 5 (4 GB), Raspberry Pi Camera Module v2 (Sony IMX219), Pi 5 kamera kablosu (22→15 pin), USB-C güç adaptörü, micro-HDMI dönüştürücü
> **Bilgisayar:** Windows 10
> **Süre:** ~2 saat

---

## İçindekiler
1. [Öğrenme hedefleri](#1-öğrenme-hedefleri)
2. [İşletim sistemini SD karta yazma](#2-işletim-sistemini-sd-karta-yazma)
3. [İlk açılış ve ağda bulma](#3-ilk-açılış-ve-ağda-bulma)
4. [SSH ile uzaktan bağlanma](#4-ssh-ile-uzaktan-bağlanma)
5. [Sistemi güncelleme ve güvenli kapatma](#5-sistemi-güncelleme-ve-güvenli-kapatma)
6. [Dosya transferi (scp)](#6-dosya-transferi-scp)
7. [Bellek: swap ve zram](#7-bellek-swap-ve-zram)
8. [Kamera kurulumu](#8-kamera-kurulumu)
9. [Yazılım kurmanın altın kuralı](#9-yazılım-kurmanın-altın-kuralı)
10. [Sıcaklık ve güç ölçümü](#10-sıcaklık-ve-güç-ölçümü)
11. [Sorun giderme](#11-sorun-giderme)
12. [Mülakat notları](#12-mülakat-notları)

---

## 1. Öğrenme hedefleri
Bu bölümün sonunda öğrenci:
- Raspberry Pi'yi **ekran kullanmadan** (headless) kurabilir ve SSH ile yönetebilir,
- Bilgisayar ile Pi arasında dosya aktarabilir,
- Linux'ta bellek (RAM / swap / zram) kavramlarını açıklayabilir ve ayarlayabilir,
- CSI kamerayı bağlayıp test edebilir, sensör modlarının görüş alanına etkisini açıklayabilir,
- Cihazın sıcaklık ve güç tüketimini ölçüp yorumlayabilir.

---

## 2. İşletim sistemini SD karta yazma

**Araç:** [Raspberry Pi Imager](https://www.raspberrypi.com/software/) (v2.0+)

| Adım | Seçim |
|---|---|
| Cihaz | Raspberry Pi 5 |
| İşletim sistemi | Raspberry Pi OS (64-bit) — Debian Trixie tabanlı, masaüstlü |
| Depolama | microSD kart (32 GB önerilir). **"Sistem sürücülerini hariç tut" işaretli kalsın.** |

**Özelleştirme (en kritik kısım):**

| Ayar | Değer |
|---|---|
| Ana makine adı (hostname) | `raspi-5` |
| Yerelleştirme | Europe/Istanbul, klavye `tr` |
| Kullanıcı | `pi` + güçlü bir şifre (not alın!) |
| Wi-Fi | SSID (büyük/küçük harf duyarlı), şifre, ülke `TR` |
| Uzaktan erişim | **SSH etkin**, şifre ile kimlik doğrulama |

> ⚠️ Yazma bittikten sonra Windows "Diski biçimlendirmeniz gerekir" uyarısı verirse **İptal** deyin. Windows, Linux bölümünü okuyamadığı için bu uyarıyı verir.

---

## 3. İlk açılış ve ağda bulma

1. SD kartı Pi'nin altındaki yuvaya takın, USB-C gücü bağlayın (Pi 5 için 5V/5A önerilir, 5V/3A da çalışır).
2. **Yeşil LED yanıp sönüyorsa** sistem SD karttan okuyor demektir. İlk açılış 3–5 dakika sürer ve Pi 1–2 kez kendiliğinden yeniden başlayabilir.
3. Bilgisayarda PowerShell açıp Pi'yi ağda arayın:

```powershell
ping raspi-5.local        # IPv6 adresi dönebilir (fe80::...)
ping -4 raspi-5.local     # IPv4 adresini zorla → örn. 192.168.1.42
```

> 💡 `.local` uzantısı **mDNS** protokolüyle çalışır: IP adresini bilmeden cihazı ismiyle bulmanızı sağlar.

---

## 4. SSH ile uzaktan bağlanma

```powershell
ssh pi@raspi-5.local
# veya
ssh pi@192.168.1.42
```

- İlk bağlantıda gelen *"Are you sure you want to continue connecting?"* sorusuna `yes` yazın (sadece `y` değil).
- Şifre yazarken ekranda **hiçbir şey görünmez**, bu normaldir.
- `pi@raspi-5:~ $` satırını görüyorsanız artık komutlar Pi'de çalışıyor demektir.

**Hangi penceredeyim?**

| Satırın başı | Komut nerede çalışır |
|---|---|
| `PS C:\Users\...>` | 💻 Bilgisayar |
| `pi@raspi-5:~ $` | 🍓 Raspberry Pi |

Pi'de IP ve Python sürümünü doğrulayın:
```bash
hostname -I          # 192.168.1.42
python3 --version    # Python 3.13.5
```

> ⚠️ **Doküman notu:** PDF'ten kopyalanan komutlarda boşluklar silinmiş olabilir (`hostname-I`, `python3--version`, `mkdir-p`, `shutdown-h`). Doğruları: `hostname -I`, `python3 --version`, `mkdir -p`, `shutdown -h`.

---

## 5. Sistemi güncelleme ve güvenli kapatma

```bash
sudo apt update && sudo apt full-upgrade -y
sudo reboot
```
- `apt update` → paket **listesini** yeniler
- `apt full-upgrade` → paketleri **yükseltir**
- `&&` → ilk komut başarılı olursa ikincisini çalıştırır

**Güvenli kapatma** (fişi doğrudan çekmek SD kartı bozabilir):
```bash
sudo shutdown -h now
```
Yeşil LED tamamen söndükten sonra gücü çekin.

---

## 6. Dosya transferi (scp)

`scp` (*secure copy*), SSH üzerinden dosya kopyalar. Komut her zaman **bilgisayardan** çalıştırılır.

```powershell
# Bilgisayar → Pi (Pi'nin ev klasörüne)
scp test.txt pi@192.168.1.42:~/

# Pi → Bilgisayar (bulunduğun klasöre; sondaki nokta önemli)
scp pi@192.168.1.42:test_2.txt .
```

> 💡 `~/` = kullanıcının ev klasörü (`/home/pi`). `scp` klasör oluşturmaz; hedef klasör önceden var olmalıdır.
> Alternatif: **FileZilla** ile `sftp://192.168.1.42` adresine sürükle-bırak.

---

## 7. Bellek: swap ve zram

### Kavramlar
- **RAM:** Programların çalışırken kullandığı hızlı bellek. Dolarsa Linux'un **OOM Killer**'ı programı sonlandırır.
- **Swap:** RAM dolunca verinin geçici olarak taşındığı alan.
- **zram:** RAM'in bir bölümünü **sıkıştırılmış** swap olarak kullanır. Hızlıdır ve SD kartı yıpratmaz, ama veriyi RAM'den gerçekten *çıkarmaz*.
- **Swap dosyası:** Veriyi SD karta yazar. Yavaştır ama RAM'i gerçekten boşaltır.

### Mevcut durumu inceleme
```bash
free -h
swapon --show
```
Örnek çıktı (Pi 5, 4 GB):
```
               total        used        free      shared  buff/cache   available
Mem:           4.0Gi       451Mi       2.9Gi        34Mi       723Mi       3.5Gi
Swap:          2.0Gi          0B       2.0Gi

NAME       TYPE      SIZE USED PRIO
/dev/zram0 partition   2G   0B  100
```
> 💡 Asıl bakılması gereken sütun **`available`**. Linux boş RAM'i disk önbelleği (`buff/cache`) olarak kullanır ve gerektiğinde anında geri verir.

### zram'ı 4 GB'a çıkarma (Pi 5)
Ana yapılandırmaya dokunmadan bir **drop-in** dosyası ekliyoruz (güncellemelerde ayarımız korunur):

```bash
sudo mkdir -p /etc/rpi/swap.conf.d
sudo nano /etc/rpi/swap.conf.d/fixedzram.conf
```
İçerik:
```ini
[Main]
Mechanism=zram+file

[Zram]
RamMultiplier=2
MaxSizeMiB=4096
```
| Parametre | Anlamı |
|---|---|
| `RamMultiplier=2` | zram en fazla fiziksel RAM'in 2 katı olabilir |
| `MaxSizeMiB=4096` | ...ama en fazla 4 GB (ikisinden küçüğü geçerli) |

**nano kısayolları:** `Ctrl+O` → Enter (kaydet), `Ctrl+X` (çık)

```bash
sudo reboot
swapon --show   # /dev/zram0 → 4G
free -h         # Swap: 4.0Gi
```

> ⚠️ **Pi Zero 2 W için tersi geçerli:** ~415 MB kullanılabilir RAM'de zram, RAM'in ~1/3'ünü kendisi tüketir. Bu yüzden ML işleri için zram kapatılıp **SD swap dosyası** kullanılmalıdır (dokümandaki ölçümde YOLO11n 320: zram ile 7562 ms, swap dosyası ile 1946 ms).

---

## 8. Kamera kurulumu

### Teori: CSI ve USB kamera
| | CSI kamera | USB kamera |
|---|---|---|
| Bağlantı | MIPI CSI-2 (doğrudan işlemciye) | USB |
| Veri | Ham (RAW / Bayer) → Pi'nin **ISP**'si işler | Kamera içinde işlenip sıkıştırılır (MJPEG) |
| Gecikme / CPU | Düşük | Daha yüksek |
| Yazılım | `libcamera` / `rpicam-*` | `fswebcam`, V4L2 |

CSI kablosunda iki yol vardır: **MIPI CSI-2** (yüksek hızlı görüntü verisi) ve **I2C** (kontrol: sensörü tanıma, pozlama, kazanç).

### Bağlantı
> ⚠️ Kamerayı **Pi kapalıyken** takın.

1. Pi 5'te **CAM/DISP 0** konnektörünün kapağını tırnakla yukarı kaldırın.
2. Kablonun **dar ucunu** (22 pin), metal temaslar konnektör pinlerine bakacak şekilde düz ve tam itin, kapağı kapatın.
3. **Geniş ucu** (15 pin) kamera kartına aynı şekilde takın.

> 💡 Pi 5'in kamera konnektörü eski Pi'lerden **daha dardır** (22 pin). Camera v2 ile kullanmak için bir ucu dar, bir ucu geniş özel kablo gerekir.

### Test
```bash
rpicam-hello --list-cameras
```
```
0 : imx219 [3280x2464 10-bit RGGB] (/base/axi/pcie@1000120000/rp1/i2c@88000/imx219@10)
    Modes: 'SRGGB10_CSI2P' : 640x480 [200.16 fps - (1000, 752)/1280x960 crop]
                             1640x1232 [81.07 fps - (0, 0)/3280x2464 crop]
                             1920x1080 [47.57 fps - (680, 692)/1920x1080 crop]
                             3280x2464 [21.19 fps - (0, 0)/3280x2464 crop]
```
- `i2c@88000/imx219@10` → Sensör, Pi 5'in RP1 çipindeki I2C hattında **0x10** adresinde tanındı.
- **RGGB** → Bayer filtresi: her piksel tek bir renk görür, tam renkli görüntüyü ISP hesaplar.
- Pi 5, IMX219'u otomatik tanır. `config.txt` dosyasına `dtoverlay=imx219,cam0` eklemek genelde **gerekmez**.

### Sensör modları ve görüş alanı (FOV)
| Mod | FPS | Görüş alanı |
|---|---|---|
| 640x480 | 200 | Merkezden **kırpılmış** (1280x960 bölge) |
| 1640x1232 | 81 | **Tam alan** (2x2 binning) |
| 1920x1080 | 47 | Merkezden kırpılmış |
| 3280x2464 | 21 | Tam alan, tam çözünürlük |

**Canlı önizleme** (ekran bağlıyken):
```bash
rpicam-hello -t 0                       # süresiz önizleme, Ctrl+C ile çık
rpicam-hello -t 0 --mode 640:480        # kırpılmış mod
rpicam-hello -t 0 --mode 1640:1232      # tam alan modu
```
> 💡 Önizlemede tüm modlar aynı akıcılıkta görünür, çünkü **ekran 60 Hz**'dir. Sistemin hızı zincirin en yavaş halkası kadardır.

**FOV deneyi:** Kamerayı oynatmadan, aynı çıktı boyutunda iki fotoğraf:
```bash
rpicam-jpeg -o crop.jpg --mode 640:480   --width 640 --height 480
rpicam-jpeg -o full.jpg --mode 1640:1232 --width 640 --height 480
```
**Sonuç:** `crop.jpg` yakın açı, `full.jpg` geniş açı. Oran ≈ 3280 / 1280 ≈ **2.56 kat**.

> 🎯 **ML için ders:** Eğitim verisini hangi kamera modunda topladıysanız canlı testi de **aynı modda** yapın. Aksi hâlde model nesneleri farklı ölçekte görür (*training/serving skew*).

---

## 9. Yazılım kurmanın altın kuralı

> ❌ Dokümandaki `sudo rm /usr/lib/python3.11/EXTERNALLY-MANAGED` komutunu **çalıştırmayın**. Hem eski (bizde Python 3.13), hem de sistem Python'unu korumasız bırakır (PEP 668).

| Ne kuruluyor? | Nasıl? |
|---|---|
| Sistem kütüphaneleri, donanım arayüzleri, derleme araçları | `sudo apt install ...` |
| Python paketleri | **Sanal ortam (venv)** içinde `pip install ...`, **sudo olmadan** |

---

## 10. Sıcaklık ve güç ölçümü

### Sıcaklık
```bash
vcgencmd measure_temp                       # temp=47.2'C
cat /sys/class/thermal/thermal_zone0/temp   # 45750 (mili-derece)
watch -n 2 vcgencmd measure_temp            # 2 saniyede bir, Ctrl+C ile çık
```
> 💡 Linux'ta "her şey bir dosyadır": donanım bilgilerinin çoğu `/sys` altında dosya olarak okunur.

### Güç (PMIC)
```bash
vcgencmd pmic_read_adc
```
Pi 5'in güç yönetim çipi her güç hattının akımını (`_A`) ve voltajını (`_V`) ölçer. Her hat için **P = V × I** hesaplanıp toplanır.

**Örnek hesap (boşta):**
| Hat | V × I | Güç |
|---|---|---|
| VDD_CORE (CPU) | 0.720 V × 1.223 A | 0.881 W |
| 1V8_SYS | 1.795 V × 0.176 A | 0.315 W |
| 0V8_SW | 0.799 V × 0.341 A | 0.272 W |
| 1V1_SYS | 1.104 V × 0.207 A | 0.228 W |
| 3V3_SYS | 3.304 V × 0.066 A | 0.219 W |
| 3V7_WL_SW (Wi-Fi) | 3.708 V × 0.048 A | 0.177 W |
| **Toplam (PMIC)** | | **≈ 2.14 W** |

PMIC toplamı USB, fan ve dönüşüm kayıplarını içermez. Kalibrasyon düzeltmesi:
**Gerçek güç ≈ 1.1451 × PMIC + 0.5879**

### Ölçüm script'i
`avg_temp_power.sh` dosyası (dokümanın s. 76–78'deki script'i, örnek sayısı 30'a indirilmiş):

```bash
#!/bin/bash
# avg_temp_power.sh - Raspberry Pi 5 ortalama sıcaklık ve güç ölçümü

SECONDS=30   # örnek sayısı (saniyede 1)

TEMP_FILE=$(mktemp)
PMIC_FILE=$(mktemp)

echo "Sampling for ${SECONDS} seconds..."

for i in $(seq 1 ${SECONDS}); do
    temp_c=$(vcgencmd measure_temp | cut -d= -f2 | cut -d"'" -f1)
    echo "$temp_c" >> "${TEMP_FILE}"

    pmic_line=$(vcgencmd pmic_read_adc)
    volts=($(echo "$pmic_line" | grep -o "volt([^)]*)=[0-9.]*V" | sed 's/.*=//;s/V//'))
    currents=($(echo "$pmic_line" | grep -o "current([^)]*)=[0-9.]*A" | sed 's/.*=//;s/A//'))

    sum_pmic=0
    for idx in "${!volts[@]}"; do
        v=${volts[$idx]}
        i_amp=${currents[$idx]}
        sum_pmic=$(awk -v a="$sum_pmic" -v b="$v" -v c="$i_amp" \
                   'BEGIN{printf "%.8f", a + b*c}')
    done

    echo "$sum_pmic" >> "${PMIC_FILE}"
    sleep 1
done

awk -v corr_a=1.1451 -v corr_b=0.5879 '
function stats(file,   x, n, sum, sum2, mean, std) {
    while ((getline x < file) > 0) { n++; sum += x; sum2 += x*x }
    close(file)
    if (n > 1) { mean = sum / n; std = sqrt((sum2 - (sum*sum)/n) / (n-1)) }
    else       { mean = sum;     std = 0 }
    return mean "|" std
}
BEGIN{
    split(stats("'"${TEMP_FILE}"'"), t, "|")
    split(stats("'"${PMIC_FILE}"'"), p, "|")
    p_real_mean = corr_a * p[1] + corr_b
    p_real_std  = corr_a * p[2]
    printf "Average_temperature = %.2f +/- %.2f °C\n", t[1], t[2]
    printf "Average_pmic_power  = %.3f +/- %.3f W\n", p[1], p[2]
    printf "Estimated_real_power = %.3f +/- %.3f W\n", p_real_mean, p_real_std
}
' /dev/null

rm -f "${TEMP_FILE}" "${PMIC_FILE}"
```
```bash
chmod +x avg_temp_power.sh    # çalıştırma izni ver
./avg_temp_power.sh
```

**Kod inceleme — zayıf noktalar (sınıfta tartışın):**
1. Voltaj ve akım hattın **adına** göre değil **sırasına** göre eşleştiriliyor (14 voltaj, 12 akım). Şu an doğru çalışıyor, ama satır sırası değişirse **sessizce yanlış** sonuç verir. → Hat adını anahtar alan bir sözlük (`declare -A`) kullanılmalı.
2. `SECONDS`, bash'te kendiliğinden artan **özel bir değişken**. → `N_SAMPLES` gibi bir isim kullanılmalı.

### Stres testi
```bash
sudo apt install stress-ng -y
# Terminal 1:
stress-ng --cpu 4 --timeout 60s
# Terminal 2 (aynı anda):
./avg_temp_power.sh
```

### Ölçüm sonuçlarımız
| Durum | Sıcaklık | PMIC gücü | Tahmini gerçek güç |
|---|---|---|---|
| Boşta | 46.88 ± 0.72 °C | 1.839 ± 0.161 W | **2.694 ± 0.185 W** |
| %100 CPU (4 çekirdek) | 57.79 ± 1.13 °C | 5.876 ± 0.249 W | **7.316 ± 0.285 W** |

**Yorum:**
- Güç ~2.7 katına çıktı: yük altında **DVFS** hem frekansı hem voltajı yükseltir (P ≈ C·V²·f).
- 57.8 °C, throttling sınırının (~80 °C) çok altında; aktif soğutma yeterli.
- Sıcaklığın sapmasının daha büyük olması, ölçüm boyunca ısının hâlâ yükseldiğini gösterir. Isıl denge için daha uzun ölçüm gerekir.

**Referans tablo (doküman, aktif soğutmalı Pi 5):**
| Durum | Sıcaklık | Güç |
|---|---|---|
| Boşta (CLI) | 40–50 °C | 3.0–3.5 W |
| Hafif masaüstü | 50–60 °C | 4–5 W |
| Orta CPU yükü | 55–70 °C | 5–7 W |
| Ağır CPU/GPU | 70–80 °C | 7–10 W |

---

## 11. Sorun giderme

| Sorun | Sebep | Çözüm |
|---|---|---|
| Imager: "depolama aygıtına yazma hatası" | Kart Imager açıkken değiştirildi / adaptör kilidi kapalı / güvenlik yazılımı | Imager'ı kapat, kartı çıkar-tak, adaptör kilidini kontrol et, Imager'ı **yönetici olarak** çalıştır. Gerekirse Windows Güvenliği → *Denetimli klasör erişimi*'ni geçici kapat. |
| `ssh : The term 'ssh' is not recognized` | OpenSSH istemcisi kurulu değil | Yönetici PowerShell: `Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0` |
| OpenSSH kurulu ama `C:\Windows\System32\OpenSSH\ssh.exe` bile bulunamıyor | **32-bit (x86) PowerShell** açılmış; Windows, System32'yi SysWOW64'e yönlendiriyor | `C:\Windows\Sysnative\OpenSSH\ssh.exe pi@192.168.1.42` veya 64-bit PowerShell kullan |
| `ssh: connect ... Connection timed out` (reboot sonrası) | Pi henüz açılmadı | 1–2 dk bekle, `ping -4 raspi-5.local` ile kontrol et |
| `raspi-config`'de *VNC Resolution* yok | Yeni Raspberry Pi OS'ta kaldırıldı | Bu adımı atla |
| PDF'ten kopyalanan komut hata veriyor | Boşluklar silinmiş | `hostname -I`, `python3 --version`, `mkdir -p`, `shutdown -h now`, `swapon --show`, `free -h` |

---

## 12. Mülakat notları

**"Linux'ta free RAM az görünüyor, sorun var mı?"**
Genelde yok. Linux boş RAM'i disk önbelleği olarak kullanır. Asıl ölçü `available` sütunudur.

**"zram her zaman iyi bir fikir mi?"**
Hayır. zram veriyi sıkıştırır ama RAM'de tutar. RAM'i çok kısıtlı cihazlarda (Pi Zero) zram'ın kendisi belleğin önemli kısmını tüketir ve OOM'a yol açabilir. **Edge AI'ın altın kuralı: daima hedef donanımda ölç.**

**"Edge vision sisteminde neden CSI kamera?"**
Ham veri doğrudan donanım ISP'sine gider. USB'deki sıkıştırma/açma ve protokol yükü yoktur: **daha düşük gecikme, daha az CPU.**

**"Training/serving skew nedir?"**
Eğitim verisi ile sahadaki verinin dağılımının farklı olması (çözünürlük, görüş alanı, ışık, kamera). Çözüm: veriyi, modelin çalışacağı donanım ve ayarlarla toplamak.

**"Sistemin FPS'ini ne belirler?"**
Pipeline'ın en yavaş halkası: *kamera → ön işleme → model → son işleme → ekran*. Edge AI'da genelde darboğaz model çıkarımıdır.

**"Neden virtual environment?"**
Bağımlılık izolasyonu: projeler birbirini ve sistemi bozmaz, ortam `requirements.txt` ile tekrarlanabilir olur.

**"Düşük voltaj neden bu kadar önemli?"**
Dinamik güç ≈ C·V²·f. Voltaj karesel etki eder; DVFS boştayken voltajı ve frekansı düşürerek enerji tasarrufu sağlar.

**"Doğru benchmark nasıl yapılır?"**
Warm-up turları at, birden çok ölçüm al, **ortalama ± standart sapma** raporla, ortam koşullarını (sıcaklık, güç modu) belirt.

**"Kod incelemede neye bakılır?"**
"Çalışıyor mu?" yetmez; **"Hangi koşulda sessizce yanlış çalışır?"** diye sor. Sıraya, sabit indekse veya tekrar kullanılan isimlere dayanan kod kırılgandır.
