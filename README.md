# SpQ DPI Bypass

> Windows kullanıcı-seviyesi DPI bypass motoru. ISP DPI sistemlerini kandırmak için TCP/TLS trafiğini modifiye eder.

## ⚡ Özellikler

| Teknik | Açıklama |
|--------|----------|
| **TLS SNI Fragmentation** | ClientHello paketini bölerek DPI'nin SNI görmesini engeller |
| **TCP Desync Injection** | Sahte paketlerle DPI durum makinesini bozar |
| **TTL Manipulation** | Sahte paketlerin sunucuya ulaşmadan ölmesini sağlar |
| **HTTP Host Split** | HTTP Host header'ını parçalara ayırır |
| **JA3 Fingerprint Spoof** | TLS parmak izini randomize eder |
| **Profil Sistemi** | Ülkeye özel ön-ayarlar (Turkey / Russia / China) |

## 🏗️ Proje Yapısı

```
SpQ-DPI/
├── core/                      # Çekirdek bypass motorları
│   ├── packet_capture.h/cpp   # WinDivert wrapper
│   ├── tls_parser.h/cpp       # TLS ClientHello parser
│   ├── http_parser.h/cpp      # HTTP request parser
│   ├── tcp_fragment.h/cpp     # TCP fragmentation engine
│   ├── desync_engine.h/cpp    # TCP desync injection
│   ├── ttl_engine.h/cpp       # TTL manipulation
│   ├── ja3_spoof.h/cpp        # JA3 fingerprint spoofer
│   ├── packet_inject.h/cpp    # Packet injection layer
│   └── evasion_engine.h/cpp   # Ana karar motoru
├── config/
│   ├── profile_manager.h/cpp  # Profil yönetimi
│   ├── profiles.json          # Profil tanımları
│   └── rules.json             # Kural tanımları
├── gui/
│   └── main_gui.h/cpp         # Win32 GUI (tray icon + kontrol panel)
├── utils/
│   ├── logger.h/cpp           # Thread-safe logger
│   └── random.h               # Rastgele sayı üretici
├── WinDivert/                 # WinDivert SDK
│   ├── include/windivert.h
│   ├── x64/                   # 64-bit DLL/SYS/LIB
│   └── x86/                   # 32-bit DLL/SYS/LIB
├── main.cpp                   # Entry point
└── CMakeLists.txt             # Build sistemi
```

## 🔧 Derleme

### Gereksinimler

- **CMake** 3.16+
- **MSVC** (Visual Studio 2019/2022) veya **MinGW-w64**
- **WinDivert** SDK (dahil)

### MSVC ile Derleme

```powershell
# Build dizini oluştur
mkdir build
cd build

# CMake yapılandır
cmake .. -G "Visual Studio 17 2022" -A x64

# Derle
cmake --build . --config Release
```

### MinGW ile Derleme

```powershell
mkdir build
cd build

cmake .. -G "MinGW Makefiles" -DCMAKE_BUILD_TYPE=Release
cmake --build .
```

### Çıktı

Build sonrası `build/Release/` (veya `build/`) dizininde:
- `SpQ_DPI_Bypass.exe` — Ana GUI uygulama
- `SpQ_DPI_Console.exe` — Konsol debug versiyonu
- `WinDivert.dll` + `WinDivert64.sys` — Driver dosyaları
- `profiles.json` + `rules.json` — Yapılandırma

## 🚀 Kullanım

1. **Yönetici olarak çalıştır** (WinDivert kernel driver gerektirir)
2. Profil seç (Turkey / Russia / China / Default)
3. İstediğin bypass tekniklerini aç/kapat
4. **START** butonuna bas
5. İnternet trafiği otomatik olarak bypass edilir

## 📊 Mimari Akış

```
[Outbound TCP Packet]
        ↓
 WinDivert Capture
        ↓
   Protocol Check
   ┌────────┬────────┐
   │ :443   │ :80    │
   ↓        ↓        ↓
  TLS     HTTP     Pass
   ↓        ↓
 Parse    Parse
ClientH  Host
   ↓        ↓
 JA3?   Fragment
   ↓
Desync?
   ↓
 TTL?
   ↓
Fragment?
   ↓
 Inject
Modified
Packets
```

## ⚠️ Uyarılar

- Bu yazılım **yalnızca eğitim amaçlıdır**
- Yasal uyumluluk tamamen kullanıcının sorumluluğundadır
- WinDivert driver yükleme hatası → **Blue Screen riski** (nadiren)
- Yönetici yetkisi gereklidir

## 📄 Lisans

Bu proje kişisel kullanım içindir. WinDivert, LGPL/GPL lisansı altındadır.
