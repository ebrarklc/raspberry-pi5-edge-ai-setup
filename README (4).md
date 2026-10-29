# 🍓 Raspberry Pi 5 — Edge AI için Sıfırdan Kurulum

**Ekran olmadan (headless) Raspberry Pi 5 kurulumu, CSI kamera, bellek optimizasyonu ve güç/sıcaklık ölçümü.**
Edge AI serisinin 1. bölümü — sonraki bölümlerde bu kurulumun üzerine görüntü sınıflandırma ve nesne tespiti modelleri çalıştırılacak.

![Raspberry Pi 5](https://img.shields.io/badge/Raspberry%20Pi-5%20(4GB)-C51A4A?logo=raspberrypi&logoColor=white)
![OS](https://img.shields.io/badge/OS-Debian%20Trixie%2064--bit-A81D33?logo=debian&logoColor=white)
![Python](https://img.shields.io/badge/Python-3.13-3776AB?logo=python&logoColor=white)
![Camera](https://img.shields.io/badge/Camera-IMX219%20v2-2E8B57)
![Dil](https://img.shields.io/badge/Dil-Türkçe-E30A17)

---

## 🎯 Bu repoda neler var?

Bu repo, Raspberry Pi 5'i **yapay zeka uygulamalarına hazır hâle getirmenin** adım adım, Türkçe ve denenmiş rehberidir.
Tüm komutlar güncel **Raspberry Pi OS (Debian Trixie, Python 3.13)** üzerinde test edilmiş; kaynak kitaptaki eski veya hatalı komutlar düzeltilmiştir.

📄 **Tam ders notu:** [01-raspberry-pi-kurulumu.md](01-raspberry-pi-kurulumu.md)

| # | Konu | Öğrenilenler |
|---|---|---|
| 1 | Headless kurulum | Raspberry Pi Imager 2.0 ile Wi-Fi + SSH ön ayarı, ekransız ilk açılış |
| 2 | Uzaktan erişim | SSH, mDNS (`raspi-5.local`), `scp` ile dosya transferi |
| 3 | Bellek | RAM / swap / zram farkı, zram'ı 2 GB → 4 GB'a çıkarma |
| 4 | Kamera | CSI ve USB farkı, IMX219 sensör modları, görüş alanı (FOV) deneyi |
| 5 | Yazılım | `apt` ve `pip` ayrımı, PEP 668, sanal ortam (venv) mantığı |
| 6 | Sıcaklık & güç | PMIC ile hat bazında güç ölçümü, stres testi, DVFS |

---

## 📊 Öne çıkan ölçüm sonuçları

### Sıcaklık ve güç (aktif soğutma)
| Durum | Sıcaklık | Tahmini gerçek güç |
|---|---|---|
| Boşta | 46.88 ± 0.72 °C | **2.69 ± 0.19 W** |
| %100 CPU (4 çekirdek, `stress-ng`) | 57.79 ± 1.13 °C | **7.32 ± 0.29 W** |

➡️ Yük altında güç **~2.7 kat** artıyor (DVFS: frekans + voltaj yükseliyor, P ≈ C·V²·f).
➡️ 57.8 °C, throttling sınırının (~80 °C) çok altında — soğutma yeterli.

### Kamera sensör modları (IMX219)
| Mod | FPS | Görüş alanı |
|---|---|---|
| 640×480 | 200 | ⚠️ Merkezden kırpılmış |
| 1640×1232 | 81 | ✅ Tam alan (2×2 binning) |
| 1920×1080 | 47 | ⚠️ Merkezden kırpılmış |
| 3280×2464 | 21 | ✅ Tam alan, tam çözünürlük |

➡️ 640×480 modunda nesneler **~2.56 kat** daha büyük görünür. Eğitim ve test verisi **aynı modda** toplanmalıdır (*training/serving skew*).

---

## 🛠️ Donanım

| Bileşen | Model |
|---|---|
| Kart | Raspberry Pi 5 (4 GB) |
| Kamera | Raspberry Pi Camera Module v2 (Sony IMX219, 8 MP) |
| Kamera kablosu | Pi 5 uyumlu (22 pin → 15 pin) |
| Güç | USB-C 5V / 5A |
| Görüntü (isteğe bağlı) | micro-HDMI dönüştürücü |

---

## ⚡ Hızlı başlangıç (özet komutlar)

```bash
# Pi'ye bağlan
ssh pi@raspi-5.local

# Güncelle
sudo apt update && sudo apt full-upgrade -y

# Bellek durumu
free -h && swapon --show

# Kamerayı listele ve canlı önizle
rpicam-hello --list-cameras
rpicam-hello -t 0

# Sıcaklık ve güç
vcgencmd measure_temp
vcgencmd pmic_read_adc
```

Ayrıntılar, açıklamalar ve ölçüm script'i için 👉 [tam ders notu](01-raspberry-pi-kurulumu.md)

---

## 🧯 Sık karşılaşılan sorunlar

| Sorun | Çözüm |
|---|---|
| Imager'da "yazma hatası" | Imager'ı kapat, kartı çıkar-tak, **yönetici olarak** çalıştır |
| Windows'ta `ssh` tanınmıyor | OpenSSH istemcisini kur; 32-bit PowerShell'de `C:\Windows\Sysnative\OpenSSH\ssh.exe` kullan |
| Kitaptaki komut hata veriyor | PDF'ten kopyalarken boşluklar silinmiş: `hostname -I`, `python3 --version`, `mkdir -p` |
| `EXTERNALLY-MANAGED` dosyasını silmek | ❌ Yapma — Python paketlerini **venv** içinde kur |

---

## 📝 Mülakat notları (özet)

- **Linux'ta "free" RAM az görünüyor?** → Normal; asıl ölçü `available` sütunu.
- **zram her zaman iyi mi?** → Hayır; RAM'i çok kısıtlı cihazlarda zararlı olabilir. *Daima hedef donanımda ölç.*
- **Neden CSI kamera?** → Ham veri doğrudan ISP'ye gider: düşük gecikme, az CPU.
- **Sistemin FPS'ini ne belirler?** → Pipeline'ın en yavaş halkası.
- **Doğru benchmark?** → Warm-up + çoklu ölçüm + ortalama ± std.

Tüm mülakat notları ders notunun sonunda.

---

## 🗺️ Edge AI serisi

| # | Bölüm | Repo |
|---|---|---|
| 1 | Raspberry Pi 5 kurulumu | 📍 *bu repo* |
| 2 | Görüntü sınıflandırma temelleri (MobileNet V2, LiteRT) | 🔜 |
| 3 | Kendi görüntü sınıflandırıcın (Edge Impulse) | 🔜 |
| 4 | Nesne tespiti (SSD MobileNet, EfficientDet) | 🔜 |

---

## 📚 Kaynak

Marcelo Rovai — *Edge AI Engineering: Hands-On with the Raspberry Pi* (UNIFEI, 2026).
Bu repo kitabın uygulamalı ve Türkçe bir takibidir.

## 👤 Hazırlayan

**Ebrar Kılıç** — Edge AI & Embedded Systems · [github.com/ebrarklc](https://github.com/ebrarklc)
