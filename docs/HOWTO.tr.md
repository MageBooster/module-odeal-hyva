# MageBooster Ödeal ödeme modülü: kullanım kılavuzu

- **Paket:** `magebooster/module-odeal`
- **Lisans:** ücretli, seri numarası gerekir
- **Ödeme yöntemi kodu:** `magebooster_odeal`
- **Destek:** info@magebooster.com

## Ne işe yarar

Bu modül Ödeal'ı kartla ödeme yöntemi olarak ekler. Ödeme, Ödeal'ın kendi 3D Secure ödeme sayfasında yapılır.

### Ödeme akışı

1. Müşteri siparişi verir. Sipariş **Pending Payment** durumunda kaydedilir. Sipariş e-postası, ödeme onaylanana kadar gönderilmez.
2. Modül Ödeal'dan bir erişim anahtarı (token) alır (`/token`).
3. Modül, sipariş numarasını `externalId` olarak kullanıp bir ödeme sayfası oluşturur (`/vpos/pay-by-link`). Müşteri bu sayfaya yönlendirilir.
4. Müşteri kartını Ödeal'ın sayfasına girer ve 3D Secure adımını tamamlar.
5. Müşteri `/mbodeal/payment/callback` adresine döner. Modül sonucu Ödeal'a sorar (`/vpos/check-status`).
   - **Ödeme başarılıysa:** fatura kesilir ve sipariş **Processing** durumuna geçer.
   - **Ödeme başarısızsa ya da müşteri vazgeçtiyse:** sipariş iptal edilir ve sepet geri yüklenir.

### Diğer davranışlar

- **Tarayıcıyı kapatan müşteriler:** `magebooster_odeal_reconcile` cron görevi her 10 dakikada bir, geri dönmeden tarayıcıyı kapatan müşterilerin siparişlerini kontrol eder.
- **Ödenmemiş siparişler:** Magento'nun kendi **Pending Payment Order Lifetime** ayarına göre iptal edilir. Bu ayar **Stores > Configuration > Sales > Sales > Orders Cron Settings** altındadır.
- **İade:** Online credit memo, yönetim panelinde tanımlı iade yolunu çağırır.
- **Dönüş adresleri:** Sipariş numarası, mağazanızın şifreleme anahtarıyla imzalanır.
- **Kart bilgisi:** Yalnızca Ödeal'ın sayfasına girilir. Mağazanız kart bilgisini hiç görmez.
- **Para birimi:** "Currencies charged directly" listesinde olmayan para birimindeki siparişler, **Stores > Currency Rates** sayfasındaki kayıtlı kurla TRY'ye çevrilir. TRY tutarı ve kur, ödeme adımında ve siparişte görünür.

## Gereksinimler

- PHP 8.0 veya üzeri
- Magento 2.4.4 – 2.4.9
- API erişimi olan bir Ödeal üye işyeri hesabı. Client ID, client secret ile test ve canlı API adreslerini Ödeal verir.
- Luma sürümü: `magebooster/module-odeal`
- Hyvä sürümü: `magebooster/module-odeal-hyva` (ayrı indirilir)

### Hyvä durumu

Bu paket, Luma tabanlı ödeme sayfaları için Knockout ödeme bileşeni içerir. Sunucu tarafı her temada aynı çalışır. README dosyasında Hyvä Checkout bileşeni "planlanan" olarak geçiyor. Hyvä mağazada kullanmadan önce Hyvä sürümünün hazır olduğunu teyit edin.

## Composer ile kurulum

```bash
composer require magebooster/module-odeal
```

## Zip dosyasıyla kurulum

Arşivi `app/code/MageBooster/Odeal` klasörüne açın, ardından aşağıdaki komutları çalıştırın.

## Modülü etkinleştirme

```bash
bin/magento module:enable MageBooster_Odeal
bin/magento setup:upgrade
bin/magento setup:di:compile
bin/magento setup:static-content:deploy -f
bin/magento cache:flush
```

Kontrol görevinin çalışması için Magento cron'unun çalışıyor olması gerekir.

## Yönetim paneli ayarları

Ayarlar **Stores > Configuration > Sales > Payment Methods > Ödeal · MageBooster** altındadır.

| Alan | Varsayılan | Açıklama |
|---|---|---|
| Enabled | Hayır | Yöntemi ödeme adımında gösterir. |
| Title | Credit / debit card (Ödeal) | Ödeme adımında ve siparişte görünen ad. |
| Text shown at checkout | (boş) | Yöntem adının altındaki kısa açıklama. |
| Test mode | Evet | Test adresini ve test bilgilerini kullanır. Gerçek para çekilmez. |
| Client ID | (boş) | Ödeal'ın verdiği istemci kimliği. |
| Client secret | (boş) | Ödeal'ın verdiği gizli anahtar. Şifreli saklanır. |
| Grant type | client_credentials | Ödeal'ın bildirdiği değer. |
| Scope | (boş) | Ödeal'ın bildirdiği değer. |
| Token path | /token | Erişim anahtarının alındığı yol. |
| Refund path | /vpos/refund | İade isteğinin gönderildiği yol. Ödeal'la teyit edin. |
| Test API address | (boş) | Ödeal bu adresi test bilgileriyle birlikte verir. |
| Live API address | (boş) | Ödeal bu adresi canlı bilgilerle birlikte verir. |
| Currencies charged directly | TRY | Virgülle ayrılmış liste. Başka para birimlerini yalnızca Ödeal hesabınızda açtıysa ekleyin. |
| Payment from applicable countries | Tüm izinli ülkeler | Yöntemi belirli ülkelerle sınırlar. |
| Payment from specific countries | (boş) | Belirli ülkeler seçildiğinde kullanılır. |
| Sort order | 62 | Ödeme yöntemleri arasındaki sırası. |
| Debug log | Hayır | İstekleri ve yanıtları `var/log/magebooster_mbodeal.log` dosyasına yazar. Kart bilgisi ve gizli bilgiler yazılmaz. |

## Test

1. **Hazırlık:** **Test mode** ve **Debug log** ayarlarını açın. Ödeal'ın verdiği test bilgilerini ve test adresini girin.
2. **Başarılı ödeme:** Test kartıyla Ödeal sayfasında ödeme yapın. Sipariş **Processing** durumuna geçmeli ve faturası kesilmeli.
3. **Tarayıcıyı kapatma:** Yeni bir sipariş verin ve Ödeal sayfasındayken tarayıcıyı kapatın.
   - Ödemeyi tamamladıysanız en geç 10 dakika içinde cron görevi siparişi güncellemeli.
   - Ödemediyseniz sipariş, Pending Payment Order Lifetime süresi dolunca iptal edilir.
4. **Vazgeçme:** Ödeal sayfasında vazgeçin. Sipariş iptal edilmeli ve sepet geri gelmeli.
5. **İade:** Ödenmiş bir sipariş için online credit memo oluşturun. İade çağrısının kayda yazıldığını kontrol edin.
6. **Canlıya geçiş:** Tüm adımlar sorunsuz geçtikten sonra Test mode'u kapatın.

## Bilinen sınırlar

- **Henüz denenmedi:** Bu sürüm Ödeal servisinde test edilmedi. Geliştirme sırasında test bilgisi yoktu.
- **Belgelenmemiş ayrıntılar:** API adresleri, token yanıtındaki alan adı ve iade yolu Ödeal'ın açık belgelerinde yer almıyor. Bu yüzden hepsi yönetim panelinden ayarlanır. Token için `accessToken`, `access_token` ve `token` alanları kabul edilir.
- **Tek geçerli token:** Her yeni token bir öncekini geçersiz kılar. Aynı saniyede çok sayıda ödeme başlarsa biri yeniden denenmek zorunda kalabilir.

## Sorun giderme

- **Yöntem görünmüyor:** **Enabled** ayarını, API adreslerini, ülke ayarlarını ve para birimi listesini kontrol edin.
- **Kayıtta token hatası:** Client ID, secret, grant type, scope ve token path değerlerini Ödeal'la teyit edin.
- **Sipariş Pending Payment'ta kalıyor:** Cron'un çalıştığını ve `/mbodeal/payment/callback` adresine dışarıdan erişilebildiğini kontrol edin.

## Kaldırma

1. **Enabled** ayarını kapatın.
2. Ödeme bekleyen sipariş kalmadığından emin olun.
3. Şu komutları çalıştırın:

```bash
bin/magento module:disable MageBooster_Odeal
composer remove magebooster/module-odeal
bin/magento setup:upgrade
bin/magento cache:flush
```

Ödeal, sahibinin tescilli markasıdır. Bu modül Ödeal tarafından yapılmamış ve onaylanmamıştır.

## Lisans ve kopyalama

Ticari lisans, bkz. `LICENSE.txt`. Telif hakkı (c) 2026 WebWorks 101 OÜ (MageBooster). Tüm hakları saklıdır.

- Bir seri numarası = bir canlı kurulum (bir mağaza alan adı). Aynı mağazanın test (staging) ve geliştirme kopyaları serbesttir.
- Her seri numarası ve her indirme, satın alan müşteriye bağlıdır. Luma ve Hyvä sürümleri ayrı ürünlerdir, seri numaraları da ayrıdır.
- Kodu kendi lisanslı mağazanız için değiştirebilirsiniz. Telif ve lisans notlarını silmeyin.
- Yasak olanlar: başka mağazalara kopyalamak, dağıtmak, yeniden satmak, alt lisans vermek, herkese açık depolarda, pazar yerlerinde veya forumlarda yayınlamak, lisans kontrolünü ya da bu notları kaldırmak, seri numarasını paylaşmak.
- İzinsiz kopyalama veya kullanımda seri numarası iade yapılmadan iptal edilir, güncelleme ve destek durur; WebWorks 101 OÜ tazminat isteyebilir ve yasal yola gidebilir. Estonya hukuku geçerlidir; yetkili mahkeme Harju İlçe Mahkemesi'dir (Harju Maakohus).

## Destek

info@magebooster.com
