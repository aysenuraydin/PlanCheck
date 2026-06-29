# PlanCheck

> İmar mevzuatına uygunluk denetimini otomatikleştiren mimari proje ön-kontrol sistemi.

PlanCheck, mimarların belediyeye sunduğu ruhsat projelerini, ilgili belediyenin imar yönetmeliği ve meclis kararları çerçevesinde **otomatik olarak denetleyen** bir uygulamadır. Manuel kontrol nedeniyle aylar süren "gönder–düzelt–tekrar gönder" döngüsünü dakikalara indirmeyi hedefler.

---

## İçindekiler

- [Problem](#problem)
- [Çözüm](#çözüm)
- [Temel Özellikler](#temel-özellikler)
- [Nasıl Çalışır](#nasıl-çalışır)
- [Sistem Mimarisi](#sistem-mimarisi)
- [Desteklenen Kontroller](#desteklenen-kontroller)
- [Kural Motoru](#kural-motoru)
- [Teknoloji Yığını](#teknoloji-yığını)
- [Kurulum](#kurulum)
- [Kullanım](#kullanım)
- [Proje Yapısı](#proje-yapısı)
- [Yol Haritası](#yol-haritası)
- [Sık Sorulan Sorular](#sık-sorulan-sorular)

---

## Problem

Mimari ruhsat süreçlerinde mimar, projesini imar yönetmeliğine ve meclis kararlarına uygun çizer, kontrol eder ve belediyeye gönderir. Belediye projeyi inceler, gözden kaçan eksikleri madde madde geri bildirir. Mimar düzeltir, tekrar gönderir, tekrar incelenir. Bu döngü, iş yoğunluğu nedeniyle **aylarca** sürebilir.

Bu süreç hem mimar hem de belediye için ciddi bir zaman ve emek kaybıdır. Kontrollerin büyük kısmı aslında **net, ölçülebilir ve kural tabanlıdır** — yani otomatikleştirilebilir.

## Çözüm

PlanCheck, projeyi belediyeye göndermeden önce (ya da belediyenin kendi tarafında) **otomatik bir ön denetimden** geçirir. Yönetmelik ve imar durumu verilerini makine tarafından okunabilir kurallara çevirir, mimari projeden ölçülen değerlerle karşılaştırır ve uygunsuzlukları gerekçesiyle birlikte raporlar.

Sonuç: el ile saatlerce süren kontrol, saniyeler içinde tamamlanır. Mimar projeyi temiz gönderir, belediye yükten kurtulur.

## Temel Özellikler

- **DXF/DWG tabanlı geometrik analiz** — Projeden çizgi, poligon, alan ve mesafe verilerini doğrudan okur; tahmin yapmaz, ölçer.
- **Belediye bazlı kural setleri** — Her belediye kendi yönetmelik ve meclis kararı yapılandırmasıyla tanımlanır.
- **İmar durumu / kotlu kroki / aplikasyon belgesi okuma** — Yüklenen resmî belgelerden (PDF/görsel) hedef değerleri OCR ile çıkarır.
- **Otomatik hesaplama** — Çekme mesafeleri, TAKS/KAKS, kat adedi, Hmax, emsal hesapları.
- **Madde madde uygunsuzluk raporu** — Her kural için ölçülen değer, beklenen değer ve sonuç.
- **Görsel işaretleme** — İhlal edilen bölgeleri proje çizimi üzerinde kırmızıyla işaretler, ölçü ve açıklama ekler.
- **Çoklu belge tutarlılık kontrolü** — İmar durumundaki değerler ile projedeki değerlerin uyumunu doğrular.

## Nasıl Çalışır

```
1. YÜKLEME
   Kullanıcı mimari projeyi (DXF) ve resmî belgeleri
   (imar durumu, kotlu kroki, bina aplikasyonu) yükler.

2. BELEDİYE SEÇİMİ
   İlgili belediye seçilir → o belediyenin kural seti yüklenir.

3. VERİ ÇIKARMA (PARSING)
   • Projeden: parsel sınırı, bina oturumu, katlar, etiketler, ölçüler
   • Belgelerden: çekme mesafeleri, TAKS, KAKS, Hmax (OCR ile)

4. HESAPLAMA
   Geometrik değerler hesaplanır (alanlar, mesafeler, oranlar).

5. DENETİM
   Hesaplanan değerler kural seti + imar durumu hedefleriyle
   karşılaştırılır.

6. RAPORLAMA
   • Madde madde metin raporu
   • Proje üzerinde görsel işaretleme (ekran görüntüleri)
```

## Sistem Mimarisi

PlanCheck katmanlı bir mimariye sahiptir. Her katman bağımsızdır; bu sayede yeni belge tipleri veya kurallar, çekirdeği değiştirmeden eklenebilir.

```
┌─────────────────────────────────────────────────────┐
│                   ARAYÜZ KATMANI                      │
│        (Dosya yükleme, belediye seçimi, rapor)        │
└───────────────────────┬───────────────────────────────┘
                        │
┌───────────────────────▼───────────────────────────────┐
│                  GİRDİ İŞLEME KATMANI                   │
│  ┌──────────────────┐      ┌────────────────────────┐  │
│  │  CAD Parser      │      │  Belge Parser (OCR)    │  │
│  │  (DXF → geometri)│      │  (PDF/görsel → değer)  │  │
│  └──────────────────┘      └────────────────────────┘  │
└───────────────────────┬────────────────────────────────┘
                        │
┌───────────────────────▼───────────────────────────────┐
│                 HESAPLAMA KATMANI                      │
│   (Alan, mesafe, oran, kesişim — Shapely geometri)     │
└───────────────────────┬────────────────────────────────┘
                        │
┌───────────────────────▼───────────────────────────────┐
│                   KURAL MOTORU                         │
│   (Belediye kural seti + imar durumu → karşılaştırma)  │
└───────────────────────┬────────────────────────────────┘
                        │
┌───────────────────────▼───────────────────────────────┐
│                 RAPORLAMA KATMANI                      │
│   (Metin raporu + görsel işaretleme + dışa aktarma)    │
└─────────────────────────────────────────────────────────┘
```

## Desteklenen Kontroller

| Kontrol | Açıklama | Kaynak |
|---------|----------|--------|
| Ön/yan/arka çekme mesafesi | Bina oturumunun parsel sınırına uzaklığı | İmar durumu |
| TAKS | Taban alanı katsayısı (oturum/parsel) | İmar durumu |
| KAKS / Emsal | Kat alanı katsayısı (toplam inşaat/parsel) | İmar durumu |
| Kat adedi | İzin verilen maksimum kat sayısı | İmar durumu / yönetmelik |
| Hmax | Maksimum bina yüksekliği | İmar durumu / yönetmelik |
| Bina derinliği | Yönetmelikte tanımlı azami derinlik | Yönetmelik |
| Saçak kotu | Kotlu krokiye göre saçak seviyesi | Kotlu kroki |
| Bina-bina mesafesi | Aynı parselde birden fazla yapıda | Yönetmelik |
| Etiket/ölçü eksikliği | Mahal isimleri, kot, ölçü çizgileri | Proje |
| Belge tutarlılığı | İmar durumu ↔ proje değer uyumu | Çapraz kontrol |

## Kural Motoru

PlanCheck'in kalbi, kuralları **koddan ayrı veri olarak** tutan kural motorudur. Bu sayede yeni belediye eklemek veya yönetmelik değişikliğine uyum sağlamak kod değişikliği gerektirmez — yalnızca yapılandırma güncellenir.

Örnek bir belediye kural seti (`belediyeler/ornek_belediye.yaml`):

```yaml
belediye: "Örnek Belediye"
yonetmelik_referans: "Planlı Alanlar İmar Yönetmeliği"
guncelleme: "2026-01-15"

kurallar:
  on_cekme:
    aciklama: "Ön bahçe çekme mesafesi"
    tip: "minimum_mesafe"
    kaynak: "imar_durumu"      # değer imar durumundan okunur
    birim: "metre"

  taks:
    aciklama: "Taban alanı katsayısı"
    tip: "maksimum_oran"
    kaynak: "imar_durumu"

  kat_adedi:
    aciklama: "İzin verilen kat adedi"
    tip: "maksimum_deger"
    kaynak: "imar_durumu"

  bina_derinligi:
    aciklama: "Azami bina derinliği"
    tip: "formul"
    formul: "0.5 * yol_genisligi + bahce_mesafesi"
    kaynak: "yonetmelik"

cad_katman_eslesmesi:
  parsel_siniri: "PARSEL"
  bina_oturumu: "BINA_OTURUM"
  cekme_cizgileri: "CEKME"
```

Motorun çalışma mantığı:

1. Seçilen belediyenin kural seti yüklenir.
2. Her kural için "ölçülen değer" hesaplama katmanından, "hedef değer" ise imar durumundan ya da yönetmelikten alınır.
3. Kuralın tipine göre karşılaştırma yapılır (`minimum_mesafe`, `maksimum_oran`, `maksimum_deger`, `formul`).
4. Sonuç bir denetim nesnesi olarak raporlama katmanına aktarılır.

## Teknoloji Yığını

| Katman | Teknoloji | Görev |
|--------|-----------|-------|
| CAD okuma | `ezdxf` | DXF dosyalarından geometri ve katman çıkarma |
| Geometri | `Shapely` | Alan, mesafe, kesişim, içinde/dışında hesapları |
| Belge okuma | `pdfplumber` / `Tesseract OCR` | PDF metni ve taranmış belgelerden değer çıkarma |
| Görselleştirme | `matplotlib` | Proje render + ihlal işaretleme |
| Kural motoru | `PyYAML` | Belediye kural setlerinin yüklenmesi |
| Arayüz | (planlanan) Web — `FastAPI` + `React` | Yükleme, seçim, rapor görüntüleme |
| DWG dönüşümü | `ODA File Converter` | DWG → DXF otomatik dönüşüm |

> Not: PlanCheck **deep learning kullanmaz.** Denetim, belirsizlik içermeyen kesin geometrik ve aritmetik hesaplara dayanır. OCR dışında yapay öğrenme gerektiren bir bileşen yoktur; OCR de hazır kütüphanelerle çözülür.

## Kurulum

```bash
# Depoyu klonla
git clone https://github.com/kullanici/plancheck.git
cd plancheck

# Sanal ortam oluştur
python -m venv venv
source venv/bin/activate      # Windows: venv\Scripts\activate

# Bağımlılıkları yükle
pip install -r requirements.txt

# (Opsiyonel) Tesseract OCR kurulumu — taranmış belgeler için
# Ubuntu:  sudo apt install tesseract-ocr tesseract-ocr-tur
# macOS:   brew install tesseract tesseract-lang
```

## Kullanım

### Komut satırından

```bash
python plancheck.py \
  --proje ornekler/proje.dxf \
  --imar-durumu ornekler/imar_durumu.pdf \
  --belediye ornek_belediye \
  --cikti rapor/
```

### Python içinden

```python
from plancheck import PlanCheck

pc = PlanCheck(belediye="ornek_belediye")
pc.proje_yukle("ornekler/proje.dxf")
pc.imar_durumu_yukle("ornekler/imar_durumu.pdf")

rapor = pc.denetle()

# Sonuçları yazdır
for madde in rapor.maddeler:
    print(f"[{madde.sonuc}] {madde.aciklama}: "
          f"ölçülen {madde.olculen}, beklenen {madde.beklenen}")

# Görsel işaretlemeleri kaydet
rapor.gorselleri_kaydet("rapor/")
```

### Örnek çıktı

```
=== PlanCheck Denetim Raporu ===
Belediye: Örnek Belediye
Proje: proje.dxf
Tarih: 2026-06-29

[UYGUN]        Ön çekme mesafesi: ölçülen 5.20m, beklenen ≥5.00m
[UYGUN DEĞİL]  Yan çekme mesafesi: ölçülen 2.80m, beklenen ≥3.00m
[UYGUN]        TAKS: ölçülen 0.38, beklenen ≤0.40
[UYGUN DEĞİL]  KAKS: ölçülen 1.85, beklenen ≤1.60
[UYGUN]        Kat adedi: ölçülen 4, beklenen ≤4

Sonuç: 2 uygunsuzluk bulundu. Detaylar ve görseller: rapor/
```

## Proje Yapısı

```
plancheck/
├── plancheck.py              # Ana giriş noktası
├── core/
│   ├── cad_parser.py         # DXF okuma ve geometri çıkarma
│   ├── belge_parser.py       # PDF/OCR ile belge okuma
│   ├── hesaplama.py          # Geometrik hesaplamalar (Shapely)
│   ├── kural_motoru.py       # Kural yükleme ve karşılaştırma
│   └── raporlama.py          # Metin raporu + görselleştirme
├── belediyeler/
│   ├── ornek_belediye.yaml   # Belediye kural setleri
│   └── ...
├── ornekler/                 # Örnek proje ve belgeler
├── rapor/                    # Üretilen raporlar (çıktı)
├── tests/
└── requirements.txt
```

## Yol Haritası

**Aşama 1 — Çekirdek (Prototip)**
- [x] DXF okuma ve katman çıkarma
- [x] Parsel/bina geometrisi tespiti
- [x] Tek kural (çekme mesafesi) hesaplama
- [x] Temel metin raporu
- [x] Render üzerinde görsel işaretleme

**Aşama 2 — Çoklu kural**
- [ ] TAKS / KAKS / kat adedi / Hmax kontrolleri
- [ ] YAML tabanlı kural motoru
- [ ] Çoklu belediye desteği

**Aşama 3 — Belge entegrasyonu**
- [ ] İmar durumu PDF okuma
- [ ] OCR ile taranmış belge desteği
- [ ] Belge ↔ proje çapraz tutarlılık kontrolü

**Aşama 4 — Platform**
- [ ] Web arayüzü (dosya yükleme, belediye seçimi)
- [ ] Kullanıcı hesapları ve proje geçmişi
- [ ] Rapor dışa aktarma (PDF)

**Aşama 5 — Ölçekleme**
- [ ] DWG doğrudan destek
- [ ] Belediye yönetim paneli (kendi kurallarını tanımlama)
- [ ] Çizim standardı şablonu ve doğrulama

## Sık Sorulan Sorular

**PlanCheck belediyenin onayının yerine mi geçiyor?**
Hayır. PlanCheck bir **ön-denetim** aracıdır. Resmî onay süreci belediyededir. Amaç, gözden kaçan net hataları erkenden yakalayarak gidip gelmeleri azaltmaktır.

**Her belediye için ayrı kurulum mu gerekiyor?**
Hayır. Kod tek; her belediye yalnızca bir kural seti (YAML) ile tanımlanır.

**Proje belli bir katman düzeninde mi çizilmeli?**
Doğru sonuç için projelerin tutarlı bir katman düzenine sahip olması gerekir. PlanCheck, belediye kural setinde tanımlanan katman eşlemesini kullanır; ileride çizim standardı şablonu ile bu garanti altına alınacaktır.

**Deep learning / yapay zekâ kullanıyor mu?**
Denetim mantığı kullanmaz — tamamen kesin geometrik ve aritmetik hesaplara dayanır. Yalnızca taranmış belgeleri okumak için hazır OCR araçları kullanılır.

---

*PlanCheck — ruhsat sürecindeki aylar süren kontrolü dakikalara indirir.*