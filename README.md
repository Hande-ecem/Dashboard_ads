# Dashboard_ads
# 📊 Digital Marketing Performance Dashboard / Dijital Pazarlama Performans Gösterge Tablosu
---

##  PROJE ÖZETİ

### 🎯 Proje Hakkında
Bu proje, **Facebook Ads** ve **Google Ads** platformlarından elde edilen dijital pazarlama verilerinin **PostgreSQL** veri tabanında birleştirilmesi ve **Looker Studio (Google Data Studio)** üzerinde etkileşimli bir performans gösterge tablosuna (dashboard) dönüştürülmesini kapsar. 

Amaç, farklı kanallardan gelen reklam harcamalarını, tıklamaları, gösterimleri ve yatırım getirisini (ROMI) tek bir merkezden takip edebilmektir.
^##Bu Projede Neler Öğrendim?
SQL & Veri Birleştirme: PostgreSQL üzerinde CTE ve UNION ALL yapılarını kullanarak iki farklı veri kaynağını (Facebook & Google) ortak bir veri modeline dönüştürmeyi öğrendim.

BI & Hesaplanan Alanlar (Calculated Fields): Looker Studio üzerinde SUM, bölme ve oranlama fonksiyonları ile dijital pazarlamanın en kritik KPI'larını (CPC, CPM, CTR, ROMI) kurguladım.

Görselleştirme & Veri Anlatıcılığı:

Çift eksenli birleşik grafik (Combo Chart) ile Ad Spend vs. ROMI eğilimlerini zaman ekseninde izlemeyi,

Çizgi grafik ile ay bazında aktif kampanya sayılarını takip etmeyi,

Isı haritalı tablolar (Table with Heatmap) ile en verimli ve en düşük performanslı kampanyaları hızlıca tespit etmeyi deneyimledim.

---

### 🛠️ Kullanılan Veri Kaynağı Sorgusu (PostgreSQL Custom Query)
Farklı tablolardan gelen Facebook ve Google Ads verilerini standart bir yapıya getirmek için aşağıdaki CTE (Common Table Expression) SQL sorgusu kullanılmıştır:

```sql
WITH faceb_cte as (
    SELECT f.ad_date,
           'facebook ads' as media_source,
           c.campaign_name,
           a.adset_name, 
           f.spend, f.impressions, f.reach, f.clicks, f.leads, f.value
    FROM facebook_ads_basic_daily f 
    LEFT JOIN facebook_adset a ON f.adset_id = a.adset_id
    LEFT JOIN facebook_campaign c ON f.campaign_id = c.campaign_id 
),
gog_ads_google_ads_birl as (
    SELECT ad_date, 
           'google ads' as media_source,
           campaign_name,
           adset_name, 
           spend, impressions, reach, clicks, leads, value
    FROM google_ads_basic_daily 

    UNION ALL 

    SELECT ad_date,
           media_source,
           campaign_name,
           adset_name, 
           spend, impressions, reach, clicks, leads, value
    FROM faceb_cte 
)
SELECT 
    ad_date,
    campaign_name, adset_name, media_source,
    SUM(spend) AS toplam_maliyet,
    SUM(impressions) AS gosterim_sayisi,
    SUM(clicks) AS tiklama_sayisi,
    SUM(value) AS toplam_value
FROM gog_ads_google_ads_birl
GROUP BY ad_date, media_source, campaign_name, adset_name;
