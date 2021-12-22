## Layihənin təsviri

- İstiqrazin Faiz Ödənilmə Tarixləri
- İstiqrazin Aliş Sifarişinin Yaradilmasi
- İstiqrazin Sifarişinin Faiz Ödənişləri

hesablayıb göstərilməsi üçün kiçik bir layihəni həyata keçirməlisiniz. 
Layihə çərçivəsində üç API yaratmalısınız.

## Tələblər
- Zəhmət olmasa müasir PHP framework-dən istifadə etməklə istiqraz layihəsini tətbiq edin.
- Hər tapşırığı bitirdikdən sonra, dəyişiklikləri commit etməyi unutmayın.
- Proyektin içərisində database yaratmaq üçün SQL script və ya migrasiya scripti olmalıdır. Layihə üçün database cədvəl strukturunu özünüz yaratmalısınız.
- Bütün PHP asılılıqlar “composer”-in köməyi ilə yüklənməlidir (bu halda composer.json repozitoriyaya əlavə etməyi unutmayın)
- Layihəni başlatmaq üçün təlimatı (INSTALL.MD) yazmağı unutmayın.
- API istifadəsi ilə bağlı misalları README.MD yazmağı unutmayın.


## Obyektlər
### İstiqraz obyekti
İstiqrazın aşağıdakı xüsusiyyətləri vardır

Xüsusiyyət | Qəbul edilən dəyər
-- | --
Emissiya tarixi | Y-m-d
Son tədavül tarixi | Y-m-d
Nominal Qiymət | digit
Kuponların ödənilmə tezliyi | 1, 2, 4, 12
Faizlərin hesablanma periodu | 360, 364, 365
Kupon faizi | 0-100

### Alış sifarişi obyekti
İstiqraz alış sifarişin aşağıdakı xüsusiyyətləri vardır: 

Xüsusiyyət | Qəbul edilən dəyər
-- | --
Sifariş tarixi | Y-m-d
Alınan istiqraz sayı | digit

## API
 
## 1. İstiqrazin Faiz Ödənilmə Tarixləri
İstiqrazın faiz ödəniş tarixlərini göstərən API hazırlamaq lazımdır.

`GET /bond/<id>/payouts`

Tarixləri müəyyən etmək üçün ilk öncə periodları hesablamaq lazımdır.

Period "Kuponların ödənilmə tezliyi" və "Faizlərin hesablanma periodu" əsasında hesablanır.

Aşağıdaki formuladan istifadə edin.

```php
switch ($FaizlərinHesablanmaPeriodu) {
    case 360:
        $periodunMüddətiniGünİlə = 12 / “Kuponların ödənilmə tezliyi” * 30;
        break;
    case 364:
        $periodunMüddətiniGünİlə = 364 / “Kuponların ödənilmə tezliyi”;
        break;
    case 365:
        $periodunMüddətiniAyİlə = 12 / “Kuponların ödənilmə tezliyi”;
        break;
}
```
Faiz Ödənilmə Tarixləri "Emissiya tarixi" və "Son tədavül tarixi" aralarında olur.

Tarixləri hesablamaq üçün "Emissiya tarixi" üzərinə period əlavə etmək lazımdır. 

Ödəniş tarixləri şənbə və bazar günləri düşməməlidir. 

Düşdüyü halda gələn bazar ertəsi istifadə olunmalıdır.


#### API cavabı aşağıdaki kimi olmalıdır:
```json
{
   "dates":[
      {
         "date":"2020-10-15"
      },
      {
         "date":"2020-11-16"
      },
      {
         "date":"2020-12-16"
      }
   ]
}
```

## 2. İstiqrazin Aliş Sifarişinin Yaradilmasi

`POST /bond/<id>/order`

**Aşağıdaki yoxlamaları təmin etmək lazımdır:**
- Alış tarixi “Emissiya tarixi”-dən  az ola bilməz.
- Alış tarix “Son tədavül tarixi”-dən cox ola bilməz.

**API aşağıdaki məlumatları POST ilə almalıdır:**
- Sifariş tarixi | Y-m-d
- Alınan istiqraz sayı | digit


## 3. İstiqrazin Sifarişinin Faiz Ödənişləri
İstiqraz sifarişin faiz ödənişlərinin məbləğlərini göstərən API hazırlamaq lazımdır.

`POST /bond/order/<order_id>`

Burada siz İstiqrazdan faiz ödənilmə tarixlərini almalısınız, daha sonra aşağıdakı hesablamanı etməlisiniz: 

`Yığılmış faizlər = (Nominal Qiymət / 100 * Kupon faizi) / Faizlərin hesablanma periodu * Keçmiş gün sayı * Sifariş sayı;`


#### Birinci periodun “Keçmiş gün sayı”  bu cür hesablanır:

Misal üçün “Sifariş tarixi”  = 14.01.2021, növbəti faiz ödənişi isə 07.02.2021, o zaman:

`“Keçmiş gün sayı” = (07.02.2021 - 14.01.2021) = 24 gün.`

#### Növbəti periodların  “Keçmiş gün sayı” bu cür hesablanır:

Misal üçün “Öncəki faiz hesablama tarixi”  = 06.02.2021, növbəti faiz ödənişi isə 08.03.2021, o zaman:

`“Keçmiş gün sayı” = (08.03.2021 - 06.02.2021) = 30 gün.`

#### API cavabı aşağıdaki kimi olmalıdır:
```json
{
   "payouts":[
      {
         "date":"2020-10-15",
         "amount":135.6164
      },
      {
         "date":"2020-11-16",
         "amount":144.6575
      },
      {
         "date":"2020-12-16",
         "amount":135.6164
      }
   ]
}
```

## Nümunə məlumatları

Siz minimum üç ədəd istiqraz və hər istiqraz üçün minimum bir ədəd sifariş yaradmalısınız.

```
Emissiya tarixi: 2021-11-08
Son tədavül tarixi	2022-11-03
Nominal Qiymət: 100
Kuponların ödənilmə tezliyi:	4
Faizlərin hesablanma periodu:	360
Kupon faizi: 10%
```

```
Emissiya tarixi: 2021-11-08
Son tədavül tarixi	2022-11-07
Nominal Qiymət: 200
Kuponların ödənilmə tezliyi:	4
Faizlərin hesablanma periodu:	364
Kupon faizi: 20%
```

```
Emissiya tarixi: 2021-11-08
Son tədavül tarixi	2022-11-08
Nominal Qiymət: 300
Kuponların ödənilmə tezliyi:	4
Faizlərin hesablanma periodu:	365
Kupon faizi: 30%
```

Sifariş xüsusiyyətləri.
```
Sifariş tarixi:2021-11-23
Sifariş sayı:10
```


