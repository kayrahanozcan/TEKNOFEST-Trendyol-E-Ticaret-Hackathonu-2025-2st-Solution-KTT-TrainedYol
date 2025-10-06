# Trendyol E-Ticaret Hackathonu 2025 - 2. Sıra Çözümü (Detaylı Teknik Açıklama)

Selam Kaggle Ailesi,

Trendyol E-Ticaret Hackathonu 2025'te 2. sırayı alan çözümümüzün teknik detaylarını sizlerle paylaşmaktan heyecan duyuyoruz. Bu yazıda, veri setine yaklaşımımızdan özellik mühendisliğine, modelleme stratejimizden elde ettiğimiz sonuçlara kadar tüm süreci adım adım açıklayacağız.

## 1. Problem ve Değerlendirme Metriği

Yarışmanın temel amacı, bir kullanıcının arama oturumu sırasında belirli bir ürüne tıklama (`clicked`) ve sipariş etme (`ordered`) olasılıklarını tahmin etmekti. Değerlendirme, `clicked` ve `ordered` hedefleri için ayrı ayrı hesaplanan AUC skorlarının ağırlıklı ortalamasıyla yapıldı:

**Final Skor = 0.2 * AUC_clicked + 0.8 * AUC_ordered**

`ordered` hedefinin ağırlığının yüksek olması, modelimizin özellikle satın alma davranışını doğru tahmin etmesinin kritik olduğunu gösteriyordu. Bu nedenle tüm stratejimizi bu metrik üzerine kurduk.

## 2. Veri Analizi ve Temel Strateji

Veri setini analiz ettiğimizde iki temel zorlukla karşılaştık:

1.  **Zamansal Uyumsuzluk:** Ana `train`/`test` verileri belirli günlerin 17:00 saatini içerirken, `content`, `user` ve `term` log verileri en son 16:00'ya kadar olan bilgileri barındırıyordu. Bu durum, veri sızıntısını (data leakage) önlemek için **t-25 saatlik bir gecikme (lag)** ile birleştirme yapmayı zorunlu kıldı. Bu, en kritik adımlarımızdan biriydi.
2.  **Hedef Dengesizliği:** `ordered` hedefinin oranı yaklaşık %0.3 gibi çok düşük bir seviyedeydi. Bu durum, modelimizin azınlık sınıfını doğru bir şekilde öğrenebilmesi için `StratifiedKFold` gibi katmanlı doğrulama yöntemlerini kullanmamızı gerektirdi.

Bu analizler doğrultusunda temel stratejimiz, **zengin tarihsel ve etkileşim özellikleri** oluşturmak ve bu özellikleri **farklı öğrenme dinamiklerine sahip iki güçlü gradient boosting modelinin (LightGBM ve CatBoost) ensemble'ı** ile beslemek oldu.

## 3. Özellik Mühendisliği (Feature Engineering)

Başarımızın anahtarı, titizlikle yürüttüğümüz özellik mühendisliği süreciydi. Tüm işlemleri, hem eğitim hem de test setine tutarlı bir şekilde uygulamak için verileri en başta birleştirdik. Veri işleme ve özellik mühendisliği adımlarında, büyük veri setleriyle verimli çalışmamızı sağlayan **Polars** kütüphanesinin **lazy execution** yeteneklerinden sonuna kadar faydalandık.

### 3.1. Kaydırma Penceresi (Rolling Window) ile Tarihsel Özellikler

Kullanıcıların, ürünlerin ve arama terimlerinin geçmiş davranışlarını yakalamak için kaydırma pencereleri kullandık. Bu amaçla, farklı log tabloları üzerinde çalışan iki ana fonksiyon geliştirdik:

*   `apply_rolling_windows_lazy_ic`: Bu fonksiyon, `total_search_impression` ve `total_search_click` metrikleri için `sum` ve `max` agregasyonları üretti.
*   `apply_rolling_windows_lazy_ccfo`: Bu fonksiyon ise `total_click`, `total_cart`, `total_fav` ve `total_order` metrikleri için `sum` ve `max` agregasyonları üretti.

Bu fonksiyonları aşağıdaki tablolara ve zaman pencerelerine uyguladık:

| Veri Seti | Gruplama Anahtarı | Zaman Penceresi | Agregasyon Metrikleri |
| :--- | :--- | :--- | :--- |
| `content_search_log` | `content_id_hashed` | `1d`, `7d`, `15d`, `30d` | Gösterim, Tıklama |
| `content_top_terms_log` | `search_term_normalized`, `content_id_hashed` | `1d`, `7d`, `15d`, `30d` | Gösterim, Tıklama |
| `term_search_log` | `search_term_normalized` | `7d`, `15d`, `30d` (Günlük) | Gösterim, Tıklama |
| `term_search_log` | `search_term_normalized` | `3h`, `12h`, `24h` (Saatlik) | Gösterim, Tıklama |
| `content_sitewide_log` | `content_id_hashed` | `1d`, `7d`, `15d`, `30d` | Tıklama, Sepet, Favori, Sipariş |
| `user_fashion_sitewide_log` | `user_id_hashed`, `content_id_hashed` | `1d`, `7d`, `15d`, `30d` | Tıklama, Sepet, Favori, Sipariş |
| `user_sitewide_log` | `user_id_hashed` | `1d`, `7d`, `15d`, `30d` | Tıklama, Sepet, Favori, Sipariş |

Bu özellikler, bir ürünün veya arama teriminin son 1, 7, 15 ve 30 gün içindeki popülerliğini ve bir kullanıcının belirli bir ürüne olan geçmiş ilgisini modelimize taşıdı.

### 3.2. Veri Birleştirme (Joining) Stratejisi

Ürettiğimiz bu tarihsel özellikleri, ana veri çerçevemizle (`df_all`) birleştirmek için dikkatli bir birleştirme stratejisi izledik. Zamansal tutarlılığı korumak ve veri sızıntısını önlemek için **t-25 saatlik gecikmeyi** burada uyguladık. Ana veri çerçevesinde `ts_date_25h` adında bir sütun oluşturduk ve log tablolarını bu sütun üzerinden birleştirdik. Birleştirme anahtarları olarak `content_id_hashed`, `user_id_hashed` ve `search_term_normalized` kullandık.

### 3.3. Diğer Özellikler

*   **Kategorik Özellikler:** `content_metadata` tablosundan gelen `level1_category_name`, `level2_category_name`, `leaf_category_name` ve `user_metadata` tablosundan gelen `user_gender` gibi özellikleri doğrudan kullandık.
*   **Sayısal Özellikler:** `content_metadata`, `content_price_data` ve `user_metadata` tablolarından gelen `attribute_type_count`, `merchant_count`, `original_price`, `selling_price`, `content_review_count`, `content_rate_avg`, `user_birth_year`, `user_tenure_in_days` gibi özellikleri modelimize dahil ettik.
*   **Türetilmiş Özellikler:**
    *   `user_age`: `user_birth_year` sütunundan kullanıcının yaşını hesapladık.
    *   `discount_rate`: `(original_price - selling_price) / original_price` formülüyle indirim oranını hesapladık.
    *   `search_term_length` ve `search_term_word_count`: `search_term_normalized` sütunundan arama teriminin uzunluğunu ve kelime sayısını çıkardık.

## 4. Model Mimarisi

Bu yarışmada, iki farklı gradient boosting kütüphanesinin gücünü birleştiren bir ensemble yaklaşımı benimsedik. `clicked` ve `ordered` hedefleri için ayrı ayrı modeller eğittik.

*   **Temel Modeller:**
    1.  **LightGBM (LGBMClassifier):** Hızı ve büyük veri setlerindeki etkinliği nedeniyle ilk tercihimizdi.
    2.  **CatBoost (CatBoostClassifier):** Kategorik özellikleri doğal olarak işlemesi ve aşırı öğrenmeye karşı dayanıklılığı ile bilinen CatBoost, model çeşitliliğimizi artırdı.

*   **Ensemble Stratejisi:**
    Her bir hedef (`clicked` ve `ordered`) için, bir **LightGBM** ve bir **CatBoost** modelinden oluşan bir **VotingRegressor** kullandık. `voting='soft'` parametresi ile modellerin olasılık tahminlerinin ortalamasını alarak daha stabil ve güçlü bir nihai tahmin elde ettik.

## 5. Eğitim ve Doğrulama (Validation) Stratejisi

*   **Çapraz Doğrulama (Cross-Validation):** Modelimizin performansını güvenilir bir şekilde ölçmek ve aşırı öğrenmeyi önlemek için **StratifiedKFold** (5 katlı) kullandık. Hedef değişkenlerdeki dengesizlik nedeniyle `StratifiedKFold`, her katmanda sınıf dağılımını koruyarak daha robust bir doğrulama sağladı.
*   **Hiperparametre Optimizasyonu:** Modellerimizin hiperparametrelerini optimize etmek için **Optuna** kütüphanesini kullandık. Optimizasyon sürecinde doğrudan yarışmanın değerlendirme metriği olan AUC'yi maksimize etmeye odaklandık.
*   **Bellek Yönetimi:** Tüm veri işleme adımlarını **Polars** ile lazy olarak gerçekleştirmek ve gereksiz değişkenleri `gc.collect()` ile temizlemek, Kaggle ortamında bellek limitlerini aşmamamızı sağladı.

## 6. Sonuç

Bu çözüm, titiz bir veri analizi, yaratıcı ve kapsamlı bir özellik mühendisliği ve güçlü bir ensemble modelleme stratejisinin birleşimidir. Özellikle, zamansal verilerle doğru çalışma (t-25 saat lag) ve farklı zaman pencerelerinde zengin tarihsel özellikler üretme yaklaşımımız, bizi liderlik tablosunda üst sıralara taşıyan temel faktörler oldu. LightGBM ve CatBoost'un güçlerini birleştirmek, tek bir modelin ulaşamayacağı bir genelleme yeteneği sağladı.

Umarız bu açıklama, yaklaşımımızı anlamanıza yardımcı olur. Herkese bol şans!
