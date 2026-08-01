# Öğrenci Sağlık Riski Tahmini / Predicting Student Health Risk

Kaggle Playground Series S6E7 yarışması için geliştirilmiş uçtan uca bir tabular machine learning projesidir.

An end-to-end tabular machine learning project developed for the Kaggle Playground Series S6E7 competition.

- [Türkçe](#türkçe)
- [English](#english)

---

## Türkçe

### Problem

Amaç, bir öğrencinin sağlık durumunu aşağıdaki üç sınıftan biri olarak tahmin etmektir:

- `at-risk`
- `fit`
- `unhealthy`

Yarışmanın değerlendirme metriği **Balanced Accuracy**’dir. Bu metrik her sınıfın recall değerini eşit önemde değerlendirir. Dolayısıyla veri setinin çoğunluğunu oluşturan `at-risk` sınıfını doğru tahmin etmek tek başına yeterli değildir.

### Yarışma sonucu

| Ölçüm | Sonuç |
|---|---:|
| OOF Balanced Accuracy | **0.950614** |
| Public leaderboard skoru | **0.95065** |
| Private leaderboard skoru | **0.95043** |
| Final sıralaması | **173** |
| Birincilik skoru | 0.95085 |
| Birincilik skoruyla fark | 0.00042 |

Yarışmanın ölçeği:

| Katılım göstergesi | Sayı |
|---|---:|
| Yarışmaya katılanlar (entrants) | 6.691 |
| Aktif katılımcılar | 3.452 |
| Takımlar | 3.355 |
| Toplam submission | 34.190 |

Bu sonuç, takım sayısına göre yaklaşık **ilk %5,2** içinde yer almaktadır.

Final leaderboard oldukça sıkışıktı. Bu nedenle çalışmada yalnızca model skoru değil; validation güvenilirliği, sınıf bazlı recall ve public/private leaderboard riski de dikkate alındı.

### Veri seti

| Veri | Satır | Sütun |
|---|---:|---:|
| Train | 690.088 | 15 |
| Test | 295.753 | 14 |

Veri setinde uyku, kalp atış hızı, BMI, egzersiz, beslenme, stres ve günlük aktiviteyle ilişkili yedi sayısal ve altı kategorik tahmin değişkeni bulunmaktadır.

Hedef değişken dengesizdir:

| Sınıf | Train oranı |
|---|---:|
| `at-risk` | %85,87 |
| `unhealthy` | %8,36 |
| `fit` | %5,77 |

Bu nedenle normal accuracy yanıltıcı olabilirdi. Proje boyunca `StratifiedKFold`, sınıf bazlı recall, OOF tahminleri ve Balanced Accuracy kullanıldı.

### İzlenen çalışma akışı

1. Train, test ve submission yapısını kontrol etme
2. Duplicate satırları, tekrar eden ID’leri ve eksik değerleri inceleme
3. Sayısal aralıkları ve IQR tabanlı outlier adaylarını değerlendirme
4. Train ve test dağılımlarını karşılaştırma
5. Sayısal ve kategorik değişkenlerin hedefle ilişkisini inceleme
6. Az sayıda, yorumlanabilir etkileşim değişkeni üretme
7. Feature-engineered CatBoost ve native kategorik LightGBM ile base ensemble oluşturma
8. Target encoding işlemini fold içinde uygulayarak hedef sızıntısını önleme
9. Üç seed ve 7-fold Target-Encoding LightGBM ile tahmin varyansını azaltma
10. Sınıf prior düzeltmesini OOF Balanced Accuracy üzerinde seçme
11. Düzeltilmiş TE modele %85, base ensemble’a %15 ağırlık verme
12. OOF raporunu, sınıf recall değerlerini ve submission yapısını doğrulama

### Feature engineering

Final notebook’ta aşağıdaki yorumlanabilir etkileşimler kullanıldı:

- Uyku süresi × BMI
- Uyku süresi × adım sayısı
- Egzersiz süresi × adım sayısı
- BMI başına uyku
- Egzersiz dakikası başına kalori
- Egzersiz dakikası başına adım
- Stres × uyku kalitesi
- Stres × fiziksel aktivite
- Uyku kalitesi × fiziksel aktivite

Her yeni feature otomatik olarak faydalı kabul edilmedi; ölçülebilir bir hipotez olarak ele alınıp cross-validation ile sınandı.

### Final model ve diğer deneyler

Final submission; feature-engineered CatBoost, LightGBM ve üç seed ile eğitilmiş
7-fold Target-Encoding LightGBM olasılıklarının prior correction sonrasında
birleştirilmesiyle üretildi.

Yarışma sürecindeki geniş model laboratuvarında ayrıca şunlar denendi:

- CatBoost, LightGBM ve XGBoost
- Target encoding kullanan LightGBM
- Multi-seed averaging
- Probability prior correction
- Sınıfa özel multiplier optimizasyonu
- Model blending ve stacking
- RealMLP
- Missing indicator değişkenleri
- Cascade ve disagreement arbiter yaklaşımları

Bu depo, ana yaklaşımı okunabilir ve yeniden üretilebilir biçimde sunar. Ayrıntılı deney günlüğü [`docs/competition-retrospective-tr.md`](docs/competition-retrospective-tr.md) dosyasındadır.

### Proje yapısı

```text
student-health-risk-kaggle/
|-- data/
|   `-- README.md
|-- docs/
|   `-- competition-retrospective-tr.md
|-- notebooks/
|   `-- final_competition_solution.ipynb
|-- .gitignore
|-- README.md
`-- requirements.txt
```

### Veriyi indirme

Yarışma verileri bu depoda yeniden dağıtılmamaktadır. Kaggle yarışma kurallarını kabul ettikten sonra verileri indirebilirsiniz:

```bash
kaggle competitions download -c playground-series-s6e7
unzip playground-series-s6e7.zip -d data
```

Yerel çalıştırma için aşağıdaki dosyaları `data/` klasörüne yerleştirin:

```text
train.csv
test.csv
sample_submission.csv
```

Notebook, Kaggle üzerinde çalıştırıldığında Kaggle input klasörünü otomatik olarak kullanır.

### Kurulum

```bash
python -m venv .venv
```

Windows:

```bash
.venv\Scripts\activate
pip install -r requirements.txt
```

macOS/Linux:

```bash
source .venv/bin/activate
pip install -r requirements.txt
```

Ardından notebook’u açıp hücreleri yukarıdan aşağıya çalıştırın.

### Temel çıkarımlar

- Validation metriği yarışma metriğiyle aynı olmalıdır.
- Dengesiz hedeflerde her sınıfın recall değeri ayrı değerlendirilmelidir.
- Model ve blend karşılaştırmaları için OOF tahminleri saklanmalıdır.
- Eksik değerler ve outlier’lar otomatik silme kuralı değil, test edilmesi gereken hipotezlerdir.
- Son sınıf kararını ayarlamadan önce model olasılıklarının kalitesi incelenmelidir.
- Küçük public leaderboard artışları yerine fold’larda tutarlı iyileşmeler tercih edilmelidir.

---

## English

### Problem

The goal is to predict a student’s health condition as one of three classes: `at-risk`, `fit`, or `unhealthy`.

The competition metric is **Balanced Accuracy**, which gives equal importance to recall for every class. Predicting the dominant `at-risk` class well is therefore not sufficient on its own.

### Competition result

| Metric | Result |
|---|---:|
| OOF Balanced Accuracy | **0.950614** |
| Public leaderboard score | **0.95065** |
| Private leaderboard score | **0.95043** |
| Final rank | **173** |
| Winning score | 0.95085 |
| Gap to winning score | 0.00042 |

Competition scale:

| Participation metric | Count |
|---|---:|
| Entrants | 6,691 |
| Participants | 3,452 |
| Teams | 3,355 |
| Total submissions | 34,190 |

Based on the number of teams, this result ranks approximately in the **top 5.2%**.

The final leaderboard was extremely close. The project therefore focused on validation reliability, class-level recall, and public/private leaderboard risk in addition to model performance.

### Dataset

| Split | Rows | Columns |
|---|---:|---:|
| Train | 690,088 | 15 |
| Test | 295,753 | 14 |

The dataset contains seven numerical and six categorical predictors related to sleep, heart rate, BMI, exercise, diet, stress, and daily activity.

The target is imbalanced:

| Class | Train share |
|---|---:|
| `at-risk` | 85.87% |
| `unhealthy` | 8.36% |
| `fit` | 5.77% |

For this reason, ordinary accuracy would be misleading. The project uses `StratifiedKFold`, class-level recall, out-of-fold predictions, and Balanced Accuracy.

### Workflow

1. Inspect the train, test, and submission structures.
2. Check duplicate rows, duplicate IDs, and missing values.
3. Review numerical ranges and IQR-based outlier candidates.
4. Compare train and test distributions.
5. Explore numerical and categorical relationships with the target.
6. Create a small set of interpretable interaction features.
7. Build a base ensemble with feature-engineered CatBoost and native-categorical LightGBM.
8. Apply target encoding inside each fold to prevent target leakage.
9. Reduce prediction variance with three seeds of 7-fold Target-Encoding LightGBM.
10. Select class-prior correction using OOF Balanced Accuracy.
11. Blend 85% corrected TE model probabilities with 15% base ensemble probabilities.
12. Validate OOF reports, class recall, and the final submission structure.

### Feature engineering

The final notebook uses the following interpretable interactions:

- Sleep duration × BMI
- Sleep duration × step count
- Exercise duration × step count
- Sleep per BMI
- Calories per exercise minute
- Steps per exercise minute
- Stress × sleep quality
- Stress × physical activity
- Sleep quality × physical activity

Each feature was treated as a testable hypothesis rather than assumed to be useful automatically.

### Final model and additional experiments

The final submission combines a feature-engineered CatBoost model, LightGBM, and
three seeds of 7-fold Target-Encoding LightGBM after probability prior correction.

The broader competition study also included CatBoost, LightGBM, XGBoost, target encoding, multi-seed averaging, prior correction, class-specific multipliers, blending, stacking, RealMLP, missing indicators, cascade models, and disagreement arbiters.

The repository presents the core workflow in a readable and reproducible form. The detailed experiment diary is available in [`docs/competition-retrospective-tr.md`](docs/competition-retrospective-tr.md).

### Repository structure

```text
student-health-risk-kaggle/
|-- data/
|   `-- README.md
|-- docs/
|   `-- competition-retrospective-tr.md
|-- notebooks/
|   `-- final_competition_solution.ipynb
|-- .gitignore
|-- README.md
`-- requirements.txt
```

### Getting the data

The competition data is not redistributed in this repository. Download it after accepting the Kaggle competition rules:

```bash
kaggle competitions download -c playground-series-s6e7
unzip playground-series-s6e7.zip -d data
```

For local execution, place `train.csv`, `test.csv`, and `sample_submission.csv` inside the `data/` directory. On Kaggle, the notebook automatically uses the Kaggle input directory.

### Installation

```bash
python -m venv .venv
```

Windows:

```bash
.venv\Scripts\activate
pip install -r requirements.txt
```

macOS/Linux:

```bash
source .venv/bin/activate
pip install -r requirements.txt
```

Then open the notebook and run the cells from top to bottom.

### Key lessons

- Match the validation metric to the competition metric.
- Evaluate each class separately when the target is imbalanced.
- Preserve OOF predictions for honest model and blend comparisons.
- Treat missing values and outliers as hypotheses, not automatic deletion rules.
- Examine probability quality before tuning the final class decision.
- Prefer stable cross-validation gains over small public leaderboard gains.
