# 🛒 Amazon E-Ticaret Verilerinde Sipariş Tutarı ve Talep Tahmini

Bu proje, Amazon e-ticaret satış verilerini kullanarak makine öğrenmesi algoritmalarıyla **ürün birim fiyatı (Amount)**, **sipariş adedi (Qty)** ve **günlük kategori bazlı toplam talep** tahminleri gerçekleştirmeyi amaçlamaktadır. Veri sızıntısını (data leakage) önleyen ön işleme adımları ve zamana dayalı eğitim-test ayrımı kullanılarak **Baseline**, **Linear Regression** ve **Random Forest** modellerinin performansları karşılaştırılmıştır.

---

## 🎯 Problem Tanımı

Projede üç temel tahmin problemi ele alınmıştır:
1. **Sipariş Tutarı Tahmini (`Amount`):** Ürün kategorisi, beden, gönderim eyaleti, B2B durumu ve takvim özellikleri kullanılarak ürün birim fiyatının (INR) tahmin edilmesi.
2. **Sipariş Miktarı Tahmini (`Qty`):** Sipariş satırındaki ürün adedinin tahmin edilmesi.
3. **Günlük Kategori Bazlı Toplam Talep Tahmini (`ToplamTalep`):** Ürün kategorisi ve takvim özellikleri kullanılarak günlük toplam satış miktarının tahmin edilmesi.

---

## 📊 Veri Seti Özeti

Projede Kaggle üzerinde yayımlanan [Amazon Sale Report (Unlock Profits with E-Commerce Sales Data)](https://www.kaggle.com/datasets/thedevastator/unlock-profits-with-e-commerce-sales-data) veri seti kullanılmıştır.

* **Orijinal Kayıt Sayısı:** 128.975
* **Temizlik Sonrası Kayıt Sayısı:** 113.701
* **Tarih Aralığı:** 31 Mart 2022 – 29 Haziran 2022
* **Para Birimi:** INR (Hindistan Rupisi)
* **Öne Çıkan Kategoriler:** `Set` (44.077 kayıt, ort. 854,47 INR), `kurta` (43.905 kayıt, ort. 465,83 INR), `Western Dress` (13.663 kayıt), `Top` (9.692 kayıt).

### Kullanılan Değişkenler
| Değişken | Açıklama | Tür |
| :--- | :--- | :--- |
| `Category` | Ürün kategorisi | Kategorik |
| `Size` | Ürün bedeni | Kategorik |
| `ship-state` | Gönderim yapılan eyalet | Kategorik |
| `B2B` | İşletmeden işletmeye satış bilgisi | Kategorik |
| `DayOfWeek` | Haftanın günü *(türetilmiş)* | Sayısal |
| `DayOfMonth` | Ayın günü *(türetilmiş)* | Sayısal |
| `IsWeekend` | Hafta sonu bilgisi *(türetilmiş)* | Sayısal |
| `Amount` | Ürün birim fiyatı *(Hedef Değişken 1)* | Sayısal |
| `Qty` | Sipariş edilen ürün miktarı *(Hedef Değişken 2)* | Sayısal |

---

## ⚙️ Metodoloji

* **Veri Temizleme ve Doğrulama:** Eksik veya sıfır/negatif `Amount`, `Qty` ve `Date` kayıtları çıkarılmıştır. Aynı SKU koduna sahip çoklu siparişler karşılaştırılarak `Amount` değişkeninin ürün birim fiyatı olduğu doğrulanmıştır.
* **Veri Sızıntısı (Data Leakage) Önlemi:** Sipariş sonrasında oluşan `Status`, `Courier Status`, `Fulfilment`, `ship-service-level` ve `fulfilled-by` sütunları modelden çıkarılmıştır. `Amount` tahmininde `Qty`, `Qty` tahmininde ise `Amount` bağımsız değişken olarak kullanılmamıştır.
* **Ön İşleme Pipeline'ı:** Kategorik değişkenler için `SimpleImputer(most_frequent)` ve `OneHotEncoder`, sayısal değişkenler için `SimpleImputer(median)` ve `StandardScaler` yapıları `ColumnTransformer` içerisinde yalnızca eğitim verisinden öğrenilecek şekilde kurgulanmıştır.
* **Zamana Dayalı Bölme (Time-Based Split):** Gelecekteki siparişleri gerçekçi simüle etmek amacıyla rastgele bölme yerine kronolojik ayrım yapılmıştır:
  * **Eğitim Seti (%81,1):** `31.03.2022 - 09.06.2022` (92.240 satır)
  * **Test Seti (%18,9):** `10.06.2022 - 29.06.2022` (21.461 satır)

---

## 📈 Model Performans Sonuçları

### 1. Sipariş Tutarı Tahmini (`Amount` - INR)
| Model | MAE (INR) | RMSE (INR) | $R^2$ |
| :--- | :---: | :---: | :---: |
| Baseline (Ortalama) | 217,190 | 276,976 | -0,006 |
| Linear Regression (`log1p/expm1`) | **152,170** | 208,431 | 0,431 |
| **Random Forest** | 152,549 | **205,692** | **0,445** |

### 2. Sipariş Miktarı Tahmini (`Qty`)
| Model | MAE | RMSE | $R^2$ |
| :--- | :---: | :---: | :---: |
| Baseline (Ortalama) | 0,0074 | 0,0661 | -0,0001 |
| Baseline (Medyan) | **0,0034** | 0,0662 | -0,0027 |
| Linear Regression | 0,0072 | **0,0660** | 0,0021 |
| Random Forest | 0,0071 | 0,0663 | -0,0067 |

*(Not: Veri setindeki siparişlerin %99,67'sinin tek ürünlük (`Qty = 1`) olmasından kaynaklanan dengesiz dağılım nedeniyle modeller varyansı açıklamada sınırlı kalmıştır.)*

### 3. Günlük Kategori Bazlı Toplam Talep Tahmini (`ToplamTalep`)
| Model | MAE | RMSE | $R^2$ |
| :--- | :---: | :---: | :---: |
| Baseline (Ortalama) | 157,644 | 175,714 | -0,031 |
| Linear Regression | 34,062 | 57,649 | 0,889 |
| **Random Forest** | **32,048** | **53,869** | **0,903** |

---

## 🛠️ Kullanılan Teknolojiler

* **Dil:** Python 3
* **Veri Analizi ve İşleme:** Pandas, NumPy
* **Makine Öğrenmesi:** Scikit-learn (`Pipeline`, `ColumnTransformer`, `TransformedTargetRegressor`, `RandomForestRegressor`, `LinearRegression`, `DummyRegressor`)
* **Görselleştirme:** Matplotlib, Seaborn
* **Geliştirme Ortamı:** Jupyter Notebook

---

## 🚀 Kurulum ve Çalıştırma

1. Depoyu bilgisayarınıza klonlayın:
   ```bash
   git clone [https://github.com/Didarum/amazon-ecommerce-ml-prediction.git](https://github.com/Didarum/amazon-ecommerce-ml-prediction.git)
   cd amazon-ecommerce-ml-prediction
