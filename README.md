# HANCI-AKINCI Kriptografik Dosya Transfer Sistemi

İki taraflı (HANCI: gönderici, AKINCI: alıcı) uçtan uca bir dosya transfer sistemi. Proje, bir BMP görüntü dosyasının bütünlük doğrulaması yapılarak, simetrik ve asimetrik şifreleme katmanlarıyla güvenli bir şekilde gönderici taraftan alıcı tarafa iletilmesini ve orijinal haliyle geri kazanılmasını amaçlar.

Bu proje, Bilişim Güvenliği Teknolojileri programı kapsamında verilen iki ödevin (Kriptografik Dosya Transfer Sistemi + GPG/PGP Güven Altyapısı) birleştirilmesiyle oluşturulmuştur.

## Mimarisi

```
HANCI (Gönderici)                              AKINCI (Alıcı)
──────────────────                             ──────────────
BMP dosyası
  → Header / Body ayrımı
  → SHA-256 hash zinciri (4 katman)
  → ZIP sıkıştırma
  → LCG dolgu (16'nın katına hizalama)
  → AES-256-CTR şifreleme
  → GPG imzalama + şifreleme    ───transfer───→  GPG deşifre + imza doğrulama
                                                  → AES-256-CTR deşifre
                                                  → Hash doğrulama (4 katman)
                                                  → Lookup tablosu ile boyut tespiti
                                                  → ZIP açma
                                                  → Orijinal BMP geri kazanımı
```

## Kullanılan Teknolojiler

| Teknoloji | Amaç |
|---|---|
| SHA-256 | Veri bütünlüğü doğrulama (4 katmanlı hash zinciri) |
| LCG (Linear Congruential Generator) | Dolgu üretimi ve boyut hizalama |
| AES-256-CTR | Simetrik şifreleme (iç katman) |
| GPG / RSA-4096 | Asimetrik şifreleme ve dijital imza (dış katman) |
| Bash + Python3 | Otomasyon ve kriptografik hesaplamalar |

## Klasör Yapısı

```
.
├── README.md
├── docs/
│   └── rehber.txt          # Adım adım uygulama rehberi
├── scripts/
│   ├── hanci.sh             # HANCI tarafı işlemleri
│   └── akinci.sh            # AKINCI tarafı işlemleri
└── .gitignore
```

## Gereksinimler

```bash
sudo apt install gnupg zip unzip openssl python3 -y
```

## Çalıştırma

Ayrıntılı adım adım rehber için `docs/rehber.txt` dosyasına bakınız. Genel akış:

1. HANCI ve AKINCI için GPG anahtar çifti üretilir, parmak izleri doğrulanır, anahtarlar karşılıklı imzalanır.
2. HANCI tarafında AES-256 anahtarı (K) üretilir ve güvenli kanal üzerinden AKINCI'ya iletilir.
3. AKINCI tarafında 800.000 satırlık SHA-256 lookup tablosu oluşturulur.
4. HANCI tarafında BMP dosyası ayrıştırılır, hashlenir, ZIP'lenir, LCG ile dolgulanır ve AES-256-CTR ile şifrelenir.
5. HANCI, şifreli dosyayı GPG ile imzalar ve AKINCI'nın açık anahtarıyla şifreler.
6. AKINCI, GPG şifresini çözer, imzayı doğrular, AES şifresini çözer, hash zincirini doğrular ve orijinal BMP dosyasını geri kazanır.

## Güvenlik Notları

Bu proje eğitim amaçlıdır. Aşağıdaki noktalar gerçek bir üretim ortamında ele alınmalıdır:

- **LCG kriptografik açıdan güvenli değildir.** Parametreleri sabit ve kamuya açıktır. Gerçek kullanımda `/dev/urandom` veya bir CSPRNG tercih edilmelidir.
- **Replay (tekrar) saldırısına karşı önlem alınmamıştır.** Paketlere zaman damgası veya nonce eklenmesi gerekir.
- **IV her iletimde yeniden üretilmemiştir.** Gerçek kullanımda her şifreleme işlemi için yeni bir IV üretilmeli ve paketin başına eklenmelidir.
- **AES anahtarı (K) düz metin dosyada saklanmıştır.** Bu yalnızca demo amaçlıdır; gerçek kullanımda anahtar yönetim sistemleri (KMS) veya güvenli bir kanal (örneğin GPG ile şifrelenmiş iletim) kullanılmalıdır.


