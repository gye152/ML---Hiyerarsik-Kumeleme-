# 🤖 Makine Öğrenmesi – Denetimsiz Öğrenme (K-Means & Hiyerarşik Kümeleme)

Bu repo, aynı veri seti üzerinde iki farklı **denetimsiz öğrenme** yöntemiyle (K-Means ve Hiyerarşik Kümeleme) yapılan segmentasyon çalışmalarını içerir. Çalışmalar Google Colab üzerinde geliştirilmiş ve sonuçlar görselleştirilmiştir.

- 📒 Notebooks:
  - `notebooks/KMeansClustering/K_Means_Clustering.ipynb`
  - `notebooks/HiyerarşikKümeleme/Hiyerarşik_kümele.ipynb`
- 🗂️ Veri seti: `flo_data_20k.csv` (aynı veri seti iki projede de kullanılmıştır)

---

## 📊 Veri Seti

- Boyut: **19.945 satır × 12 sütun**
- Bazı sütunlar:
  - `master_id`, `order_channel`, `last_order_channel`
  - **Tarih alanları:** `first_order_date`, `last_order_date`, `last_order_date_online`, `last_order_date_offline`
  - **Sipariş/harcama:** `order_num_total_ever_online`, `order_num_total_ever_offline`, `customer_value_total_ever_online`, `customer_value_total_ever_offline`
  - `interested_in_categories_12` (müşterinin ilgi duyduğu kategoriler)
- Ön işleme sırasında tarih alanları `datetime` tipine çevrilmiştir.

---

## 🛠️ Özellik Mühendisliği (Her iki çalışma için ortak)

Veriden segmentasyon için kullanılacak sayısal özellikler türetilmiştir:

- **Toplam sipariş ve harcama**
  - `order_num_total = online + offline`
  - `customer_value_total = online + offline`
- **Recency:** `today_date - last_order_date` (gün)
- **Tenure:** `last_order_date - first_order_date` (gün)
- **Purchase Frequency:** `order_num_total / (tenure + 1)`
- **Intensity:** `order_num_total / (recency + 1)`

📉 Aykırı değerleri azaltmak için sayısal sütunlarda **Winsorize (%5 alt, %5 üst)** uygulanmıştır.  
⚖️ Ölçekleme:
- K-Means için **MinMaxScaler**,
- Hiyerarşik için **StandardScaler** kullanılmıştır.

---

## 🔹 1) K-Means Clustering

**Amaç:** Optimum k değerini belirleyip müşteri segmentlerini çıkarmak.

- **Ölçekleme:** `MinMaxScaler`
- **Optimum K:** **Elbow** (Yellowbrick `KElbowVisualizer`) ve Silhouette analizi ile **k = 6**
- **Model:** `KMeans(n_clusters=6, random_state=42)`
- **Görselleştirme:** PCA ile 2 boyuta indirgenmiş dağılım grafikleri
  - PCA açıklanan varyans: **PC1 ≈ %42,2**, **PC2 ≈ %24,4**  
    (Toplam ≈ **%66,6**; iki bileşen verinin üçte ikisini açıklar.)
- **Genel gözlem (PCA üzerinden):**
  - **Segment 0** ve **Segment 1** net ayrışan gruplar.
  - **Segment 2–3–4** kısmen örtüşüyor (benzer davranış kalıpları).
  - **Segment 5** en geniş ve dağılımı yüksek grup.

> ℹ️ Not: Segment numaraları istatistiksel etiketlerdir; anlamları, özet istatistik ve görsellerle yorumlanır.

---

## 🔹 2) Hiyerarşik Kümeleme (Agglomerative)

**Amaç:** Hiyerarşik yapı üzerinden küme sayısını dendrogram ile inceleyip kesim yapmak.

- **Ölçekleme:** `StandardScaler`
- **Linkage:** `ward` (Öklid uzaklığı, varyansı minimize eder)
- **Dendrogram:** küme yapısı görselleştirilmiştir.
- **Kesim:** `fcluster(..., criterion="maxclust", t=4)` ile **4 küme**
- **Özet ve yorum (özet istatistik + görseller):**
  - **Cluster 1:** Yüksek değerli ama alışveriş sıklığı düşük → yeniden aktifleştirilmeli.
  - **Cluster 2:** En değerli → aktif & sadık müşteriler → korunmalı.
  - **Cluster 3:** Yeni müşteriler, düşük harcama → geliştirilmeye açık.
  - **Cluster 4:** Uzun süredir alışveriş yapmayan, düşük değerli → kaybedilmiş müşteri profili.

> ℹ️ Not: Küme numaraları istatistikseldir; anlamlandırma, ölçütlerin (recency, tenure, value, frequency…) kümeye göre ortalamalarıyla yapılır.

---

## ⚙️ Kullanılan Teknolojiler

- Python, NumPy, Pandas
- Scikit-learn (KMeans, AgglomerativeClustering, StandardScaler/MinMaxScaler, PCA)
- SciPy (hierarchical `linkage`, `dendrogram`, `fcluster`, `winsorize`)
- Matplotlib, Seaborn
- Yellowbrick (Elbow görselleştirme)

---
