# Trendyol E-Ticaret Hackathonu 2025 - 2. Sıra Çözümü (Detaylı Teknik Açıklama)

Yarışma: https://www.teknofest.org/tr/yarismalar/e-ticaret-hackathonu/
Problem
Bu yarışmada katılımcılar, Trendyol'un moda ile ilişkili popüler arama verilerini ve kullanıcı davranışlarını kullanarak arama sonuçlarının tıklama ve satın alıma dönüşüm oranını arttırmak üzerine çalışacaklar.

Amaç: Belirli bir kullanıcı, arama terimi ve tarih (bu üçlü kombinasyonu bir oturum olarak değerlendiriyoruz) için kullanıcının her ürüne:

Tıklayıp tıklamayacağını (clicked)
Sipariş verip vermeyeceğini (ordered)
göz önünde bulundurarak bu aksiyonların alındığı ürünleri öne çıkaracak sıralamalar üretmektir.

Değerlendirme
Çözümlerin değerlendirilmesinde üretilen sıralamalar üzerinde sipariş ve tıklama durumları için ayrı ayrı AUC metriği hesaplanacaktır. AUC, rastgele seçilen bir pozitif örneğin (tıklanan/sipariş verilen ürün), rastgele seçilen bir negatif örnekten (tıklanmayan/sipariş verilmeyen ürün) sıralamada daha önce görünme olasılığını temsil edecektir. Bu bağlamda 1'e yakın AUC değerleri, modelin ilgili ürünleri listenin üst sıralarına yerleştirme konusunda daha başarılı olduğunu gösterecektir.

Tıklama ve sipariş AUC değerleri, yalnızca ilgili aksiyonların (tıklama/sipariş) gerçekleştiği oturumlar için ayrı ayrı hesaplanacak ve her bir metrik için bu oturumların ortalaması alınacaktır. Nihai skorlamada sipariş tahminlerinin doğruluğu, tıklama tahminlerine göre daha yüksek ağırlığa sahiptir. Final skor, bu ortalama AUC değerlerinin ağırlıklı toplamı olarak hesaplanacaktır:


Burada S_click tıklama aksiyonu içeren oturumlar kümesini, S_order ise sipariş aksiyonu içeren oturumlar kümesini temsil etmektedir.

Tahmin Formatı
Test setindeki her oturum (session id) için, ürünleri (content ID) sipariş ve tıklama olasılıklarına göre sıralamanız gerekmektedir. Bu değerler boşlukla ayrılmış formatta olmalıdır. Her satır bir oturumu ifade eder ve her oturum için ürettiğiniz sıralamalarda oturumdaki bütün ürünleri kullanmanız gerekmektedir. Submission dosyası sütun isimlerini içermeli ve aşağıdaki formatta olmalıdır:

session_id,prediction
test_4fd3705b497bbe4f,564e63f4540cd721 d2ce51cef87f18c6 ff2753a5b6fa6ee8...
test_e717abe75e333f76,972474c123473e31 14e6061e74c6ee22 feb28f583317d7db...

# Trendyol E-Ticaret Hackathonu 2025 - Veri Açıklaması

Bu belge, Trendyol E-Ticaret Hackathonu 2025 yarışmasında kullanılan veri setlerinin detaylı açıklamasını sunmaktadır. Veri setleri, kullanıcıların arama oturumları, ürün bilgileri ve etkileşim geçmişleri gibi zengin bilgiler içermektedir.

## Önemli Notlar

*   **Hashlenmiş Kimlikler**: Tüm kullanıcı ve içerik kimlikleri hashlenmiş ve anonimleştirilmiş formattadır.
*   **Anonimleştirilmiş Değerler**: Fiyat ve ham aksiyon (tıklama, satın alım, sepete atma vb.) sayıları, çözümlerin performansını fazla etkilemeyecek miktarda gürültülendirilmiş ve ortak bir katsayı seti kullanılarak `[0,1]` aralığında ölçeklendirilmiştir. Bu değerlerin anonimleştirilmiş olması, etkileşim tipi (toplama, çıkarma, çarpma, bölme vb.) featureları üretmenizi engellemeyecektir.
*   **Eksik Değerler**: Bazı kolonlarda null değerler bulunabilir.

## Ana Veri Setleri

### Oturum Verileri

#### `train_sessions.parquet`

**Açıklama**: Eğitim verisi - kullanıcı arama oturumları ve etkileşim bilgileri.

| Kolon Adı              | Veri Tipi | Açıklama                                       |
| :--------------------- | :-------- | :--------------------------------------------- |
| `ts_hour`              | `datetime`  | Oturum zamanı (saat bazında)                   |
| `search_term_normalized` | `string`    | Normalize edilmiş arama terimi                 |
| `clicked`              | `int64`     | Ürüne tıklanma durumu (0: tıklanmadı, 1: tıklandı) |
| `ordered`              | `int64`     | Ürün sipariş durumu (0: sipariş verilmedi, 1: sipariş verildi) |
| `added_to_cart`        | `int64`     | Sepete ekleme durumu (0: eklenmedi, 1: eklendi) |
| `added_to_fav`         | `int64`     | Favorilere ekleme durumu (0: eklenmedi, 1: eklendi) |
| `user_id_hashed`       | `string`    | Hashlenmiş kullanıcı kimliği                   |
| `content_id_hashed`    | `string`    | Hashlenmiş ürün kimliği                        |
| `session_id`           | `string`    | Oturum kimliği                                 |

#### `test_sessions.parquet`

**Açıklama**: Test verisi - tahmin yapılacak oturum verileri.

| Kolon Adı              | Veri Tipi | Açıklama                       |
| :--------------------- | :-------- | :----------------------------- |
| `ts_hour`              | `datetime`  | Oturum zamanı (saat bazında)   |
| `search_term_normalized` | `string`    | Normalize edilmiş arama terimi |
| `user_id_hashed`       | `string`    | Hashlenmiş kullanıcı kimliği   |
| `content_id_hashed`    | `string`    | Hashlenmiş ürün kimliği        |
| `session_id`           | `string`    | Oturum kimliği                 |

### Ürün Verileri

#### `content/metadata.parquet`

**Açıklama**: Ürün kategori bilgileri ve özellik sayıları.

| Kolon Adı                      | Veri Tipi            | Açıklama                                                                |
| :----------------------------- | :------------------- | :---------------------------------------------------------------------- |
| `level1_category_name`         | `string`             | Birincil kategori adı (örn: Giyim, Ayakkabı)                            |
| `level2_category_name`         | `string`             | İkincil kategori adı (örn: Üst Giyim, Spor Ayakkabı)                    |
| `leaf_category_name`           | `string`             | Uç kategori adı (en spesifik kategori)                                  |
| `attribute_type_count`         | `float64`            | Ürün özellik türü sayısı                                                |
| `total_attribute_option_count` | `float64`            | Toplam özellik seçeneği sayısı                                          |
| `merchant_count`               | `float64`            | Ürünü satan satıcı sayısı                                               |
| `filterable_label_count`       | `float64`            | Filtrelenebilir özellik sayısı                                          |
| `content_creation_date`        | `datetime[μs, UTC]`  | Ürün oluşturulma tarihi                                                 |
| `cv_tags`                      | `string`             | Ürün görselleri kullanılarak yapay zeka ile üretilmiş ilgili olabilecek terimler |
| `content_id_hashed`            | `string`             | Hashlenmiş ürün kimliği                                                 |

#### `content/price_rate_review_data.parquet`

**Açıklama**: Ürün fiyat geçmişi, değerlendirme ve yorum bilgileri.

| Kolon Adı                      | Veri Tipi | Açıklama                                   |
| :----------------------------- | :-------- | :----------------------------------------- |
| `update_date`                  | `datetime`  | Güncelleme tarihi                          |
| `original_price`               | `float64`   | Orijinal fiyat                             |
| `selling_price`                | `float64`   | Satış fiyatı                               |
| `discounted_price`             | `float64`   | İndirimli fiyat                            |
| `content_review_count`         | `float64`   | Ürün yorum sayısı                          |
| `content_review_wth_media_count` | `float64`   | Görsel içeren yorum sayısı                 |
| `content_rate_count`           | `float64`   | Ürün değerlendirme sayısı                  |
| `content_rate_avg`             | `float64`   | Ortalama ürün puanı                        |
| `content_id_hashed`            | `string`    | Hashlenmiş ürün kimliği                    |

#### `content/search_log.parquet`

**Açıklama**: Ürün bazında arama gösterim ve tıklama verileri.

| Kolon Adı                 | Veri Tipi | Açıklama                                       |
| :------------------------ | :-------- | :--------------------------------------------- |
| `date`                    | `datetime`  | Tarih                                          |
| `total_search_impression` | `float64`   | Toplam arama gösterimi sayısı (`[0,1]` aralığında) |
| `total_search_click`      | `float64`   | Toplam arama tıklaması sayısı (`[0,1]` aralığında) |
| `content_id_hashed`       | `string`    | Hashlenmiş ürün kimliği                        |

#### `content/sitewide_log.parquet`

**Açıklama**: Ürün bazında site geneli etkileşim verileri.

| Kolon Adı           | Veri Tipi | Açıklama                                       |
| :------------------ | :-------- | :--------------------------------------------- |
| `date`              | `datetime`  | Tarih                                          |
| `total_click`       | `float64`   | Toplam tıklama sayısı (`[0,1]` aralığında)         |
| `total_cart`        | `float64`   | Toplam sepete ekleme sayısı (`[0,1]` aralığında)   |
| `total_fav`         | `float64`   | Toplam favorilere ekleme sayısı (`[0,1]` aralığında) |
| `total_order`       | `float64`   | Toplam sipariş sayısı (`[0,1]` aralığında)         |
| `content_id_hashed` | `string`    | Hashlenmiş ürün kimliği                        |

#### `content/top_terms_log.parquet`

**Açıklama**: Ürün bazında en popüler arama terimleri.

| Kolon Adı                 | Veri Tipi | Açıklama                                       |
| :------------------------ | :-------- | :--------------------------------------------- |
| `date`                    | `datetime`  | Tarih                                          |
| `search_term_normalized` | `string`    | Normalize edilmiş arama terimi                 |
| `total_search_impression` | `float64`   | Toplam arama gösterimi sayısı (`[0,1]` aralığında) |
| `total_search_click`      | `float64`   | Toplam arama tıklaması sayısı (`[0,1]` aralığında) |
| `content_id_hashed`       | `string`    | Hashlenmiş ürün kimliği                        |

### Kullanıcı Verileri

#### `user/metadata.parquet`

**Açıklama**: Kullanıcı demografik bilgileri.

| Kolon Adı            | Veri Tipi | Açıklama                       |
| :------------------- | :-------- | :----------------------------- |
| `user_gender`        | `string`    | Kullanıcı cinsiyeti            |
| `user_birth_year`    | `float64`   | Kullanıcı doğum yılı          |
| `user_tenure_in_days` | `int64`     | Kullanıcı üyelik süresi (gün cinsinden) |
| `user_id_hashed`     | `string`    | Hashlenmiş kullanıcı kimliği   |

#### `user/search_log.parquet`

**Açıklama**: Kullanıcı bazında arama davranışları.

| Kolon Adı                 | Veri Tipi | Açıklama                                       |
| :------------------------ | :-------- | :--------------------------------------------- |
| `ts_hour`                 | `datetime`  | Zaman damgası (saat bazında)                   |
| `total_search_impression` | `float64`   | Toplam arama gösterimi sayısı (`[0,1]` aralığında) |
| `total_search_click`      | `float64`   | Toplam arama tıklaması sayısı (`[0,1]` aralığında) |
| `user_id_hashed`          | `string`    | Hashlenmiş kullanıcı kimliği                   |

#### `user/sitewide_log.parquet`

**Açıklama**: Kullanıcı bazında site geneli aktiviteler.

| Kolon Adı           | Veri Tipi | Açıklama                                       |
| :------------------ | :-------- | :--------------------------------------------- |
| `ts_hour`           | `datetime`  | Zaman damgası (saat bazında)                   |
| `total_click`       | `float64`   | Toplam tıklama sayısı (`[0,1]` aralığında)         |
| `total_cart`        | `float64`   | Toplam sepete ekleme sayısı (`[0,1]` aralığında)   |
| `total_fav`         | `float64`   | Toplam favorilere ekleme sayısı (`[0,1]` aralığında) |
| `total_order`       | `float64`   | Toplam sipariş sayısı (`[0,1]` aralığında)         |
| `user_id_hashed`    | `string`    | Hashlenmiş kullanıcı kimliği                   |

#### `user/top_terms_log.parquet`

**Açıklama**: Kullanıcı bazında moda kategorisiyle ilişkili arama terimlerine ait geçmiş.

| Kolon Adı                 | Veri Tipi | Açıklama                                       |
| :------------------------ | :-------- | :--------------------------------------------- |
| `ts_hour`                 | `datetime`  | Zaman damgası (saat bazında)                   |
| `search_term_normalized` | `string`    | Normalize edilmiş arama terimi                 |
| `total_search_impression` | `float64`   | Toplam arama gösterimi sayısı (`[0,1]` aralığında) |
| `total_search_click`      | `float64`   | Toplam arama tıklaması sayısı (`[0,1]` aralığında) |
| `user_id_hashed`          | `string`    | Hashlenmiş kullanıcı kimliği                   |

#### `user/fashion_search_log.parquet`

**Açıklama**: Kullanıcı bazında moda kategorisiyle ilişkili arama logları.

| Kolon Adı                 | Veri Tipi | Açıklama                                       |
| :------------------------ | :-------- | :--------------------------------------------- |
| `ts_hour`                 | `datetime`  | Zaman damgası (saat bazında)                   |
| `total_search_impression` | `float64`   | Toplam arama gösterimi sayısı (`[0,1]` aralığında) |
| `total_search_click`      | `float64`   | Toplam arama tıklaması sayısı (`[0,1]` aralığında) |
| `user_id_hashed`          | `string`    | Hashlenmiş kullanıcı kimliği                   |
| `content_id_hashed`       | `string`    | Hashlenmiş ürün kimliği                        |

#### `user/fashion_sitewide_log.parquet`

**Açıklama**: Kullanıcı bazında moda kategorisiyle ilişkili ürünlere ait site geneli aktiviteler.

| Kolon Adı           | Veri Tipi | Açıklama                                       |
| :------------------ | :-------- | :--------------------------------------------- |
| `ts_hour`           | `datetime`  | Zaman damgası (saat bazında)                   |
| `total_click`       | `float64`   | Toplam tıklama sayısı (`[0,1]` aralığında)         |
| `total_cart`        | `float64`   | Toplam sepete ekleme sayısı (`[0,1]` aralığında)   |
| `total_fav`         | `float64`   | Toplam favorilere ekleme sayısı (`[0,1]` aralığında) |
| `total_order`       | `float64`   | Toplam sipariş sayısı (`[0,1]` aralığında)         |
| `user_id_hashed`    | `string`    | Hashlenmiş kullanıcı kimliği                   |
| `content_id_hashed` | `string`    | Hashlenmiş ürün kimliği                        |

### Arama Terimi Verileri

#### `term/search_log.parquet`

**Açıklama**: Arama terimi bazında genel aksiyon istatistikleri.

| Kolon Adı                 | Veri Tipi | Açıklama                                       |
| :------------------------ | :-------- | :--------------------------------------------- |
| `ts_hour`                 | `datetime`  | Zaman damgası (saat bazında)                   |
| `search_term_normalized` | `string`    | Normalize edilmiş arama terimi                 |
| `total_search_impression` | `float64`   | Toplam arama gösterimi sayısı (`[0,1]` aralığında) |
| `total_search_click`      | `float64`   | Toplam arama tıklaması sayısı (`[0,1]` aralığında) |



Dataset Description
Önemli Notlar
Hash'lenmiş Kimlikler: Tüm kullanıcı ve içerik kimlikleri hash'lenmiş ve anonimleştirilmiş formattadır.
Anonimleştirilmiş Değerler: Fiyat ve ham aksiyon (tıklama, satın alım, sepete atma vb.) sayıları çözümlerin performansını fazla etkilemeyecek miktarda gürültülendirilmiş ve ortak bir katsayı seti kullanılarak [0,1] aralığında ölçeklendirilmiştir. Bu değerlerin anonimleştirilmiş olması etkileşim tipi (toplama, çıkarma, çarpma, bölme vb.) featureları üretmenizi engellemeyecektir.
Eksik Değerler: Bazı kolonlarda null değerler bulunabilir.
Ana Veri Setleri
Oturum Verileri
train_sessions.parquet
Açıklama: Eğitim verisi - kullanıcı arama oturumları ve etkileşim bilgileri

Kolon Adı	Veri Tipi	Açıklama
ts_hour	datetime	Oturum zamanı (saat bazında)
search_term_normalized	string	Normalize edilmiş arama terimi
clicked	int64	Ürüne tıklanma durumu (0: tıklanmadı, 1: tıklandı)
ordered	int64	Ürün sipariş durumu (0: sipariş verilmedi, 1: sipariş verildi)
added_to_cart	int64	Sepete ekleme durumu (0: eklenmedi, 1: eklendi)
added_to_fav	int64	Favorilere ekleme durumu (0: eklenmedi, 1: eklendi)
user_id_hashed	string	Hash'lenmiş kullanıcı kimliği
content_id_hashed	string	Hash'lenmiş ürün kimliği
session_id	string	Oturum kimliği
test_sessions.parquet
Açıklama: Test verisi - tahmin yapılacak oturum verileri

Kolon Adı	Veri Tipi	Açıklama
ts_hour	datetime	Oturum zamanı (saat bazında)
search_term_normalized	string	Normalize edilmiş arama terimi
user_id_hashed	string	Hash'lenmiş kullanıcı kimliği
content_id_hashed	string	Hash'lenmiş ürün kimliği
session_id	string	Oturum kimliği
Ürün Verileri
content/metadata.parquet
Açıklama: Ürün kategori bilgileri ve özellik sayıları

Kolon Adı	Veri Tipi	Açıklama
level1_category_name	string	Birincil kategori adı (örn: Giyim, Ayakkabı)
level2_category_name	string	İkincil kategori adı (örn: Üst Giyim, Spor Ayakkabı)
leaf_category_name	string	Uç kategori adı (en spesifik kategori)
attribute_type_count	float64	Ürün özellik türü sayısı
total_attribute_option_count	float64	Toplam özellik seçeneği sayısı
merchant_count	float64	Ürünü satan satıcı sayısı
filterable_label_count	float64	Filtrelenebilir özellik sayısı
content_creation_date	datetime[μs, UTC]	Ürün oluşturulma tarihi
cv_tags	string	Ürün görselleri kullanılarak yapay zeka ile üretilmiş ilgili olabilecek terimler
content_id_hashed	string	Hash'lenmiş ürün kimliği
content/price_rate_review_data.parquet
Açıklama: Ürün fiyat geçmişi, değerlendirme ve yorum bilgileri

Kolon Adı	Veri Tipi	Açıklama
update_date	datetime	Güncelleme tarihi
original_price	float64	Orijinal fiyat
selling_price	float64	Satış fiyatı
discounted_price	float64	İndirimli fiyat
content_review_count	float64	Ürün yorum sayısı
content_review_wth_media_count	float64	Görsel içeren yorum sayısı
content_rate_count	float64	Ürün değerlendirme sayısı
content_rate_avg	float64	Ortalama ürün puanı
content_id_hashed	string	Hash'lenmiş ürün kimliği
content/search_log.parquet
Açıklama: Ürün bazında arama gösterim ve tıklama verileri

Kolon Adı	Veri Tipi	Açıklama
date	datetime	Tarih
total_search_impression	float64	Toplam arama gösterimi sayısı ([0,1] aralığında)
total_search_click	float64	Toplam arama tıklaması sayısı ([0,1] aralığında)
content_id_hashed	string	Hash'lenmiş ürün kimliği
content/sitewide_log.parquet
Açıklama: Ürün bazında site geneli etkileşim verileri

Kolon Adı	Veri Tipi	Açıklama
date	datetime	Tarih
total_click	float64	Toplam tıklama sayısı ([0,1] aralığında)
total_cart	float64	Toplam sepete ekleme sayısı ([0,1] aralığında)
total_fav	float64	Toplam favorilere ekleme sayısı ([0,1] aralığında)
total_order	float64	Toplam sipariş sayısı ([0,1] aralığında)
content_id_hashed	string	Hash'lenmiş ürün kimliği
content/top_terms_log.parquet
Açıklama: Ürün bazında en popüler arama terimleri

Kolon Adı	Veri Tipi	Açıklama
date	datetime	Tarih
search_term_normalized	string	Normalize edilmiş arama terimi
total_search_impression	float64	Toplam arama gösterimi sayısı ([0,1] aralığında)
total_search_click	float64	Toplam arama tıklaması sayısı ([0,1] aralığında)
content_id_hashed	string	Hash'lenmiş ürün kimliği
Kullanıcı Verileri
user/metadata.parquet
Açıklama: Kullanıcı demografik bilgileri

Kolon Adı	Veri Tipi	Açıklama
user_gender	string	Kullanıcı cinsiyeti
user_birth_year	float64	Kullanıcı doğum yılı
user_tenure_in_days	int64	Kullanıcı üyelik süresi (gün cinsinden)
user_id_hashed	string	Hash'lenmiş kullanıcı kimliği
user/search_log.parquet
Açıklama: Kullanıcı bazında arama davranışları

Kolon Adı	Veri Tipi	Açıklama
ts_hour	datetime	Zaman damgası (saat bazında)
total_search_impression	float64	Toplam arama gösterimi sayısı ([0,1] aralığında)
total_search_click	float64	Toplam arama tıklaması sayısı ([0,1] aralığında)
user_id_hashed	string	Hash'lenmiş kullanıcı kimliği
user/sitewide_log.parquet
Açıklama: Kullanıcı bazında site geneli aktiviteler

Kolon Adı	Veri Tipi	Açıklama
ts_hour	datetime	Zaman damgası (saat bazında)
total_click	float64	Toplam tıklama sayısı ([0,1] aralığında)
total_cart	float64	Toplam sepete ekleme sayısı ([0,1] aralığında)
total_fav	float64	Toplam favorilere ekleme sayısı ([0,1] aralığında)
total_order	float64	Toplam sipariş sayısı ([0,1] aralığında)
user_id_hashed	string	Hash'lenmiş kullanıcı kimliği
user/top_terms_log.parquet
Açıklama: Kullanıcı bazında moda kategorisiyle ilişkili arama terimlerine ait geçmiş

Kolon Adı	Veri Tipi	Açıklama
ts_hour	datetime	Zaman damgası (saat bazında)
search_term_normalized	string	Normalize edilmiş arama terimi
total_search_impression	float64	Toplam arama gösterimi sayısı ([0,1] aralığında)
total_search_click	float64	Toplam arama tıklaması sayısı ([0,1] aralığında)
user_id_hashed	string	Hash'lenmiş kullanıcı kimliği
user/fashion_search_log.parquet
Açıklama: Kullanıcı bazında moda kategorisiyle ilişkili arama logları

Kolon Adı	Veri Tipi	Açıklama
ts_hour	datetime	Zaman damgası (saat bazında)
total_search_impression	float64	Toplam arama gösterimi sayısı ([0,1] aralığında)
total_search_click	float64	Toplam arama tıklaması sayısı ([0,1] aralığında)
user_id_hashed	string	Hash'lenmiş kullanıcı kimliği
content_id_hashed	string	Hash'lenmiş ürün kimliği
user/fashion_sitewide_log.parquet
Açıklama: Kullanıcı bazında moda kategorisiyle ilişkili ürünlere ait site geneli aktiviteler

Kolon Adı	Veri Tipi	Açıklama
ts_hour	datetime	Zaman damgası (saat bazında)
total_click	float64	Toplam tıklama sayısı ([0,1] aralığında)
total_cart	float64	Toplam sepete ekleme sayısı ([0,1] aralığında)
total_fav	float64	Toplam favorilere ekleme sayısı ([0,1] aralığında)
total_order	float64	Toplam sipariş sayısı ([0,1] aralığında)
user_id_hashed	string	Hash'lenmiş kullanıcı kimliği
content_id_hashed	string	Hash'lenmiş ürün kimliği
Arama Terimi Verileri
term/search_log.parquet
Açıklama: Arama terimi bazında genel aksiyon istatistikleri

Kolon Adı	Veri Tipi	Açıklama
ts_hour	datetime	Zaman damgası (saat bazında)
search_term_normalized	string	Normalize edilmiş arama terimi
total_search_impression	float64	Toplam arama gösterimi sayısı ([0,1] aralığında)
total_search_click	float64	Toplam arama tıklaması sayısı ([0,1] aralığında)



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
