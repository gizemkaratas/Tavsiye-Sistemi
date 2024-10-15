# Armut Hizmet Tavsiye Sistemi

Bu proje, Türkiye'nin en büyük online hizmet platformlarından biri olan Armut için **Birliktelik Kuralı Öğrenimi (Association Rule Learning)** kullanarak bir ürün tavsiye sistemi geliştirmeyi amaçlamaktadır. Armut, hizmet verenlerle hizmet almak isteyenleri bir araya getirerek, temizlik, tadilat ve nakliyat gibi hizmetlerin kolayca bulunmasını sağlamaktadır. Kullanıcıların aldıkları hizmetler üzerinden, birlikte sıkça alınan hizmetleri tavsiye eden bir sistem geliştirilmiştir.

## İş Problemi

Armut, kullanıcıların platform üzerinden temizlik, tadilat, nakliyat gibi hizmetlere kolayca erişmelerini sağlar. Kullanıcıların birlikte satın alma eğiliminde oldukları hizmetleri **Birliktelik Kuralı Öğrenimi** kullanarak tavsiye eden bir sistem oluşturmak hedeflenmektedir.

## Veri Seti

Veri seti, müşterilerin aldıkları hizmetler ve bu hizmetlerin kategorilerinden oluşmaktadır. Her hizmetin satın alındığı tarih ve saat bilgisi mevcuttur. Veri setinin ana özellikleri:

- `UserId`: Müşteri numarası.
- `ServiceId`: Anonimleştirilmiş hizmet ID’si. Aynı `ServiceId` farklı kategorilerde farklı hizmetleri ifade edebilir.
- `CategoryId`: Anonimleştirilmiş kategori ID’si.
- `CreateDate`: Hizmetin satın alındığı tarih ve saat bilgisi.

### Veri Seti Örneği:
| UserId | ServiceId | CategoryId | CreateDate  |
|--------|-----------|------------|-------------|
| 1      | 5         | 8          | 2020-01-12  |
| 2      | 7         | 4          | 2020-01-15  |

## Proje Adımları

### 1. Veri Hazırlama
- `armut_data.csv` dosyası `pandas` kullanılarak okunur.
- `ServiceId` ve `CategoryId` birleştirilerek her hizmeti benzersiz bir şekilde temsil eden `Hizmet` adlı yeni bir değişken oluşturulur.
- `CreateDate` sütunundan sadece **yıl ve ay** bilgilerini içeren yeni bir tarih değişkeni oluşturulur ve bu değişken ile `UserId` birleştirilerek her kullanıcı için sepet (basket) tanımı yapılır (`SepetID`).

### 2. Sepet Temsilinin Oluşturulması
- Veri seti, her satırın `SepetID` ve her sütunun hizmet (`Hizmet`) olduğu bir **pivot tablo** haline getirilir. Tablo değerleri, hizmetin ilgili sepette olup olmadığını göstermek için 0 ve 1'lerden oluşur.

### 3. Birliktelik Kuralı Öğrenimi
- **apriori** algoritması kullanılarak minimum %1 destek değeri ile sık hizmet kümeleri bulunur.
- **association_rules** fonksiyonu ile birliktelik kuralları oluşturulur ve `support` metriği kullanılır.

### 4. Tavsiye Sistemi Fonksiyonu
- `arl_recommender()` fonksiyonu, bir ürün (hizmet) ID'si alır ve lift değerine göre en uygun hizmetleri tavsiye eder.

```python
def arl_recommender(rules_df, product_id, rec_count=1):
    sorted_rules = rules_df.sort_values("lift", ascending=False)
    recommendation_list = []
    for i, product in sorted_rules["antecedents"].items():
        for j in list(product):
            if j == product_id:
                recommendation_list.append(list(sorted_rules.iloc[i]["consequents"]))
    recommendation_list = list({item for item_list in recommendation_list for item in item_list})
    return recommendation_list[:rec_count]
****
