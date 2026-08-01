# Kaggle Playground S6E7 — Predicting Student Health Risk

Bu not, sadece “yarışmada ne yaptık?” raporu değil. Daha sonra geri dönüp baktığımızda **nasıl düşündüğümüzü**, **neyi doğru yaptığımızı**, **nerede zaman kaybettiğimizi** ve **bir sonraki yarışmada hangi sırayla ilerlememiz gerektiğini** hatırlatacak bir öğrenme günlüğüdür.

---

## 1. Yarışma özeti

Problem: Öğrencilerin sağlık durumunu üç sınıftan birine tahmin etmek.

- `at-risk`
- `fit`
- `unhealthy`

Veri tipi: Tabular veri.

Ana feature grupları:

- Uyku süresi
- Kalp ritmi
- BMI
- Kalori harcaması
- Adım sayısı
- Egzersiz süresi
- Su tüketimi
- Diyet tipi
- Stres seviyesi
- Uyku kalitesi
- Fiziksel aktivite seviyesi
- Sigara/alkol
- Cinsiyet

Yarışma metriği: **Balanced Accuracy**

Bu metrik her sınıfın recall değerini ayrı ayrı hesaplayıp ortalamasını alır. Bu yüzden sadece çoğunluk sınıfını iyi tahmin etmek yetmez.

---

## 2. Final sonucumuz

Final private leaderboard:

- Private score: yaklaşık `0.95043`
- Rank: `173`
- Winner score: yaklaşık `0.95085`
- Winner ile fark: yaklaşık `0.00042`

Bu fark çok küçük. Bu yarışmada temel modelleme tarafında doğru yoldaydık. Üst sıralar daha çok şu alanlarda ayrıştı:

- Daha fazla base model
- Daha fazla seed
- Daha iyi target encoding varyantları
- Class-specific ensemble weighting
- LogLoss + Balanced Accuracy iki aşamalı optimizasyon
- Public leaderboard yerine OOF disiplinine bağlı kalmak

Ana çıkarım:

> Bu yarışmada farkı “tek bir daha iyi model” değil, olasılıkları daha iyi birleştirme ve karar sınırlarını metriğe göre ayarlama belirledi.

---

## 3. İlk bakınca neyi anlamamız gerekiyordu?

Bu veri setine ilk baktığımızda cevaplamamız gereken sorular şunlardı:

1. Bu sınıflandırma mı, regresyon mu?
2. Hedef değişken kaç sınıflı?
3. Hedef sınıflar dengeli mi?
4. Yarışma metriği ne?
5. Train/test ayrımı rastgele mi, zaman bazlı mı, group bazlı mı?
6. ID sızıntı taşıyor mu?
7. Eksik değerler rastgele mi, yoksa bilgi taşıyor mu?
8. Kategorik feature'lar hedefle güçlü ilişkili mi?
9. Sayısal feature'larda mantıksız değer veya outlier var mı?
10. Baseline ne kadar güçlü?

Bu yarışmada en kritik iki cevap:

- Hedef dengesizdi.
- Metrik balanced accuracy idi.

Bu iki bilgi validation ve model kararlarımızı belirledi.

---

## 4. Biz nasıl düşündük?

### 4.1 Önce metriği anladık

Balanced accuracy yüzünden normal accuracy takip etmek anlamsızdı.

Örneğin sürekli `at-risk` tahmini yapan model normal accuracy'de yüksek görünebilir ama balanced accuracy'de kötü olur.

Doğru refleks:

```text
Eğer hedef dengesizse:
accuracy değil,
class recall + balanced accuracy takip et.
```

### 4.2 Hedef dağılımına baktık

Sınıf oranları yaklaşık:

- `at-risk`: %85.87
- `unhealthy`: %8.36
- `fit`: %5.77

Bu bize şunu söyledi:

- `at-risk` tahmini kolay.
- `fit` ve `unhealthy` recall değerlerini yükseltmek asıl mesele.
- Modelin raw argmax kararları çoğunluk sınıfına fazla yaslanabilir.

### 4.3 Validation'ı StratifiedKFold yaptık

Doğru yaptığımız şeylerden biri buydu.

Neden?

Çünkü her fold içinde sınıf oranları korunmalıydı. Eğer fold'lardan birinde minority class oranı saparsa validasyon skoru güvenilmez olurdu.

Bu yarışmada doğru validation:

```text
StratifiedKFold + Balanced Accuracy
```

### 4.4 OOF probability sakladık

Bu çok önemliydi.

OOF prediction/probability saklamasaydık şunları yapamazdık:

- Threshold tuning
- Class multiplier tuning
- Probability blending
- Stacking
- Fold bazlı güvenlik analizi
- Error analysis

Bir sonraki yarışmada kural:

> Her ciddi model için mutlaka OOF probability ve test probability kaydedilecek.

---

## 5. EDA tarafında ne yaptık?

Yaptığımız kontroller:

- Train/test shape
- Duplicate row
- Duplicate ID
- Eksik değerler
- Hedef dağılımı
- Sayısal feature summary
- Kategorik feature unique değerleri
- Train/test kategorik uyumu
- Sayısal feature'ların hedefe göre ortalama/medyan farkı
- Kategorik feature'ların hedef sınıf oranları

İyi yaptığımız taraf:

- Hedef dengesizliğini erken fark ettik.
- `stress_level`, `sleep_duration`, `sleep_quality`, `physical_activity_level` gibi güçlü sinyalleri gördük.
- Eksik değerlerin oranlarını train/test karşılaştırdık.
- Kategorik değerlerin train/test içinde uyumlu olduğunu kontrol ettik.

Zaman kaybettiğimiz taraf:

- Bazı EDA bölümlerini gereğinden fazla uzattık.
- Model kararını değiştirmeyecek grafiklerde fazla zaman harcadık.

Bir sonraki yarışma için EDA kuralı:

> EDA'nın amacı güzel grafik üretmek değil, modelleme kararını etkileyecek bilgi bulmaktır.

Minimum EDA checklist:

1. Shape
2. Target distribution
3. Missing summary
4. Duplicate / ID kontrolü
5. Numeric describe
6. Kategorik unique ve train/test uyumu
7. Target'a göre 3-5 güçlü ilişki kontrolü

Bu yeterli değilse derinleşilir.

---

## 6. Feature engineering tarafında ne yaptık?

Yaptığımız feature'lar:

- `sleep_bmi_interaction`
- `sleep_step_interaction`
- `exercise_step_interaction`
- `sleep_per_bmi`
- `calorie_per_exercise`
- `steps_per_exercise`
- `stress_sleep_quality`
- `stress_activity`
- `sleep_quality_activity`

Bu feature'ları neden düşündük?

Çünkü sağlık riski tek değişkenle değil, değişkenlerin beraberliğiyle açıklanabilir.

Örnek düşünce:

```text
Uyku süresi tek başına önemli.
Ama uyku süresi + stres seviyesi daha anlamlı olabilir.
```

Başka örnek:

```text
Adım sayısı yüksek ama egzersiz süresi düşükse bu farklı bir profil olabilir.
Egzersiz süresi yüksek ama adım sayısı düşükse başka bir profil olabilir.
```

Öğrendiğimiz şey:

> Feature engineering rastgele sütun üretmek değildir. Değişkenler arasında anlamlı ilişki kurmaktır.

Bu yarışmada feature engineering küçük katkı verdi ama asıl sıçrama calibration ve ensemble tarafından geldi.

---

## 7. Modelleme tarafında ne yaptık?

### 7.1 CatBoost

CatBoost iyi başlangıç modeliydi.

Neden?

- Kategorik değişkenleri doğal işleyebiliyor.
- Missing değerleri yönetebiliyor.
- Tabular yarışmalarda güçlü baseline.

Denemeler:

- Unweighted CatBoost
- `auto_class_weights="SqrtBalanced"`
- Farklı depth değerleri
- Feature engineering eklenmiş CatBoost
- Probability multiplier ile düzeltilmiş CatBoost

Öğrendiğimiz:

> Class weighting minority recall'u artırır ama majority recall'u düşürür. Balanced accuracy için bu trade-off çoğu zaman kabul edilebilir.

### 7.2 LightGBM

LightGBM tek başına her zaman en iyi değildi ama ensemble içinde katkı verdi.

Öğrendiğimiz:

> Tek başına daha zayıf model, hataları farklıysa ensemble içinde değerlidir.

### 7.3 XGBoost

XGBoost da ayrı hata paterni verdi. Bazı blendlerde katkı sağladı.

Özellikle XGBoost ile TE LightGBM disagreement analizinde bazı satırlarda XGBoost'un daha doğru olduğu görüldü. Ama arbiter denemesi güvenli submission koşullarını sağlamadı.

### 7.4 Target Encoding

Target encoding ciddi katkı verdi.

Ama burada en önemli kural:

> Target encoding tüm train üzerinde yapılırsa leakage olur. Mutlaka fold içinde yapılmalı.

Bu yarışmada TE LightGBM güçlü adaylardan biri oldu.

---

## 8. Probability multiplier ve calibration tarafında ne öğrendik?

Balanced accuracy için raw probability argmax yeterli değildi.

Sebep:

- Majority class çok baskın.
- Model doğal olarak `at-risk` sınıfına fazla güveniyor.
- Minority class recall değerleri düşüyor.

Çözüm:

```text
probability * class_multiplier
```

Yani modelin ürettiği olasılıkları sınıf bazında çarparak karar sınırını değiştiriyoruz.

Bu yarışmada çok kritik öğrendiğimiz şey:

> Modeli değiştirmeden sadece karar mekanizmasını metriğe göre ayarlamak ciddi skor kazandırabilir.

Bir sonraki yarışma için kural:

- Binary classification ise threshold tuning.
- Multiclass ise class multiplier tuning.
- F1, balanced accuracy gibi metriklerde raw argmax'a körü körüne güvenme.

---

## 9. Ensemble tarafında ne yaptık?

Denediğimiz ensemble türleri:

- CatBoost + LightGBM
- CatBoost depth varyantları
- TE LightGBM + base model
- XGBoost + TE LightGBM + base model
- Multi-seed TE LightGBM
- Logistic regression stack
- Weighted probability blend
- Disagreement / arbiter denemeleri

İyi yaptığımız taraf:

- OOF üzerinden blend ağırlığı seçtik.
- Public LB'ye körü körüne uymadık.
- Prediction distribution kontrol ettik.
- Fold bazlı improvement baktık.

Zayıf kaldığımız taraf:

- Base model havuzumuz 2. sıra çözümü kadar geniş değildi.
- Class-specific ensemble weight optimization yapmadık.
- SLSQP / Nelder-Mead gibi optimizasyonları sınırlı kullandık.

---

## 10. Ne çalıştı?

En çok katkı verenler:

- CatBoost `SqrtBalanced`
- Target encoded LightGBM
- Multi-seed TE LightGBM + base blend
- Class multiplier / prior correction
- OOF üzerinden seçim yapmak
- Fold stability kontrolü

Pratik ders:

> Bu yarışmada ham model gücünden çok OOF probability ile ne yaptığımız önemliydi.

---

## 11. Ne çok katkı vermedi?

Beklediğimiz kadar katkı vermeyenler:

- Basit missing indicator feature'ları
- CatBoost depth 8
- RealMLP full OOF'ta beklenen katkıyı vermedi
- Arbiter/disagreement modelleri OOF'ta küçük artış verse de güvenli değildi
- Public skor için agresif ayarlar private açısından riskliydi

Bu da önemli:

> Çalışmayan deneme de bilgi üretir. Ama çalışmayan şeyi final çözüme zorla eklememek gerekir.

---

## 12. Yaptığımız hatalar / iyileştirilecek davranışlar

### Hata 1 — Bazı aşamalarda fazla detayda boğulduk

EDA önemliydi ama bazı kontroller model kararını değiştirmedi.

Bir sonraki yarışmada:

```text
Önce baseline.
Sonra EDA'yı baseline hatalarına göre derinleştir.
```

### Hata 2 — Deneyleri baştan daha sistematik adlandırmalıydık

Bir süre sonra notebook içinde hangi değişkenin hangi modele ait olduğu karıştı.

Bir sonraki yarışmada:

- `oof_catboost_v1`
- `test_catboost_v1`
- `oof_lgb_te_seed42`
- `test_lgb_te_seed42`

gibi net isimlendirme yapılmalı.

### Hata 3 — Checkpoint sistemi baştan kurulmalıydı

Uzun eğitimlerden sonra kernel restart riski oluştu.

Bir sonraki yarışmada:

```text
Her modelden sonra OOF ve test probability .npy olarak kaydedilecek.
```

### Hata 4 — Score lab ile ana notebook ayrılmalıydı

Ana notebook paylaşılabilir olmalı.

Score lab ise deneme alanı olmalı.

Doğru yapı:

```text
01_eda.ipynb
02_clean_solution.ipynb
03_score_lab.ipynb
04_final_submission.ipynb
```

### Hata 5 — Üst seviye çözümleri daha erken okumalıydık

Yarışma sırasında iyi public notebook ve discussion'ları daha erken takip etmek gerekir.

Ama kopyalamak için değil:

> Hangi fikirlerin işe yaradığını görmek için.

---

## 13. İyi yaptıklarımız

- Metrik odaklı düşündük.
- Hedef dengesizliğini erken fark ettik.
- StratifiedKFold kullandık.
- Class recall değerlerini takip ettik.
- OOF probability sakladık.
- Public LB yerine OOF'a güvenmeye çalıştık.
- Çalışmayan denemeleri final çözüme zorla eklemedik.
- Son private skor public skora yakın kaldı; bu validation'ın tamamen kötü olmadığını gösterdi.

En önemli iyi refleks:

> Yeni fikirleri public LB'ye göre değil, OOF + fold stability + prediction distribution ile değerlendirdik.

---

## 14. 2. sıra çözümünden öğrendiklerimiz

2. sıra çözümü 18 base predictor kullandı:

- LightGBM
- XGBoost
- CatBoost
- FT-Transformer
- RealMLP
- HGBC

Ana farklar:

### 14.1 Class-specific ensemble weighting

Biz çoğu yerde modele tek ağırlık verdik.

2. sıra çözümünde her modelin her sınıf için ayrı ağırlığı optimize edildi:

```text
18 model x 3 sınıf = 54 ağırlık
```

Bu daha güçlüdür çünkü:

- Bir model `fit` sınıfında iyi olabilir.
- Başka bir model `unhealthy` sınıfında iyi olabilir.
- Başka bir model `at-risk` sınıfında daha kalibre olabilir.

Bir sonraki yarışmada denenmeli:

```text
class_specific_weight[model, class]
```

### 14.2 Önce LogLoss, sonra Balanced Accuracy

2. sıra çözümü iki aşamalı ilerledi:

1. SLSQP ile OOF LogLoss optimize edildi.
2. Nelder-Mead ile class multiplier tuning yapıldı.

Bu çok iyi bir prensip:

> Önce probability kalitesini iyileştir, sonra metriğe göre karar sınırını ayarla.

### 14.3 Public LB gürültüsünü doğru yorumladılar

Public leaderboard azınlık sınıflarda gürültülü olabilir.

Bu yüzden:

- Public skoru kovalamadılar.
- Full OOF üzerinde optimizasyon yaptılar.
- Teste aynı ayarları uyguladılar.

Bu yarışma için en büyük derslerden biri bu.

---

## 15. Bir sonraki tabular yarışmada yol haritamız

### Aşama 1 — Problem okuma

Sorulacak sorular:

- Hedef ne?
- Metrik ne?
- Veri tabular mı, text mi, image mı, time series mi?
- Train/test ayrımı nasıl?
- External data serbest mi?
- Submission formatı ne?

### Aşama 2 — Minimum EDA

Bakılacaklar:

- Shape
- Target distribution
- Missing summary
- Duplicate
- ID kontrolü
- Numeric describe
- Kategorik unique
- Train/test drift kontrolü

### Aşama 3 — Baseline

İlk hedef:

```text
Hızlı OOF skoru almak.
```

Tabular için:

- CatBoost
- LightGBM
- XGBoost

Text için:

- TF-IDF + Logistic Regression
- Sonra transformer

Time series için:

- Naive baseline
- Lag/rolling feature
- Time-based validation

### Aşama 4 — Validation güvenliği

Kural:

- Dengesiz sınıf: StratifiedKFold
- Group leakage: GroupKFold
- Zaman serisi: TimeSeriesSplit / forward validation
- Aynı kullanıcı/session varsa: Group split

### Aşama 5 — OOF sistemi

Her model için kaydedilecek:

- OOF probability
- Test probability
- Fold scores
- Prediction distribution
- Model config

### Aşama 6 — Feature engineering

Önce yorumlanabilir feature:

- Interaction
- Ratio
- Difference
- Count
- Frequency
- Target encoding
- Missing indicators

Ama her feature OOF ile test edilmeli.

### Aşama 7 — Calibration / threshold

Metrik türüne göre:

- F1: threshold tuning
- Balanced accuracy: class multiplier
- LogLoss: probability calibration
- AUC: ranking kalitesi
- RMSE/MAE: regression residual analizi

### Aşama 8 — Ensemble

Sıra:

1. Basit average
2. Weighted average
3. Class-specific weighted average
4. Stacking
5. Calibration sonrası blend

### Aşama 9 — Final seçim

Sadece en yüksek public skor seçilmez.

Bakılacaklar:

- OOF score
- Fold stability
- Public score
- Prediction distribution
- Model diversity
- Private risk

---

## 16. Bu yarışmayı CV/GitHub'da nasıl anlatırım?

English:

> Built a tabular machine learning pipeline for Kaggle Playground Series S6E7: Predicting Student Health Risk. The project covers EDA, stratified cross-validation, CatBoost, LightGBM, XGBoost, target encoding, class imbalance handling, probability multiplier tuning, and ensemble experimentation. Finished 173rd on the private leaderboard, within roughly 0.00042 of the winning score.

Türkçe:

> Kaggle Playground Series S6E7 Predicting Student Health Risk yarışması için EDA, StratifiedKFold validation, CatBoost, LightGBM, XGBoost, target encoding, class imbalance yönetimi, probability multiplier tuning ve ensemble denemelerini içeren uçtan uca bir tabular ML projesi geliştirdim. Private leaderboard'da 173. sırada tamamladım ve birincilik skoruna yaklaşık 0.00042 farkla yaklaştım.

---

## 17. Kısa sonuç

Bu yarışmada öğrendiğimiz ana şey:

> Tabular yarışmalarda iyi skor sadece iyi model kurmak değildir. Sağlam validation, OOF disiplini, probability calibration, model çeşitliliği ve metrik odaklı karar mekanizması birlikte gerekir.

Bir sonraki yarışmada hedefimiz:

```text
Daha erken baseline.
Daha temiz notebook.
Daha sistematik checkpoint.
Daha geniş model havuzu.
Class-specific ensemble optimization.
Public LB yerine OOF disiplini.
```

