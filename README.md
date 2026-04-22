# GLiNER_train

Bu repo Azərbaycan dili üçün NER məlumatını GLiNER formatına çevirmək və GLiNER modeli üzərində ilkin fine-tuning təcrübələri aparmaq üçün hazırlanıb. Əsas iş axını [GLINER.ipynb](/Users/nazrinaliyeva/Desktop/GLiNER_train-1/GLINER.ipynb) notebook-u içindədir.

## Layihənin məqsədi

Notebook aşağıdakı mərhələləri həyata keçirir:

- Hugging Face üzərindən `LocalDoc/azerbaijani-ner-dataset` datasetini yükləyir
- `tokens` və `ner_tags` sütunlarını string formatından siyahıya çevirib təmizləyir
- Uyğunsuz və ya pozulmuş sətrləri filtrdən keçirir
- Dataseti `train`, `validation`, `test` hissələrinə bölür
- Nəticəni GLiNER-in gözlədiyi JSON formatına çevirir
- Tam label xəritəsi və sadələşdirilmiş label xəritəsi ilə təcrübələr aparır
- `urchade/gliner_multi-v2.1` və `urchade/gliner_small-v2.1` modelləri ilə təlim yoxlamaları edir

## Repo strukturu

- [GLINER.ipynb](/Users/nazrinaliyeva/Desktop/GLiNER_train-1/GLINER.ipynb): bütün preprocessing, çevirmə və training sınaqları
- [README.md](/Users/nazrinaliyeva/Desktop/GLiNER_train-1/README.md): layihə təsviri
- `.gitignore`: lokal checkpoint və model artefaktlarını Git-dən kənarda saxlayır

## Dataset haqqında

Notebook-da istifadə olunan ilkin dataset:

- Mənbə: `LocalDoc/azerbaijani-ner-dataset`
- İlkin `train` ölçüsü: `99,545` sətir
- Təmizləmədən sonra qalan sətir sayı: `95,953`
- Buraxılan sətrlər: `3,592`

Təmizləmədən sonra split belə qurulur:

- `train`: `86,357`
- `validation`: `4,798`
- `test`: `4,798`

## Label-lar

Notebook-da əvvəlcə geniş label xəritəsi istifadə olunur. Buraya `PERSON`, `LOCATION`, `ORGANISATION`, `DATE`, `TIME`, `MONEY`, `FACILITY`, `PRODUCT`, `EVENT`, `LAW`, `LANGUAGE`, `GPE`, `NORP`, `CARDINAL`, `QUANTITY`, `POSITION`, `PROJECT` və başqa siniflər daxildir.

Daha sonra GLiNER üçün sadələşdirilmiş bir versiya da hazırlanır. Bu versiyada əsas saxlanılan label-lar:

- `PERSON`
- `LOCATION`
- `ORGANISATION`
- `DATE`
- `TIME`
- `MONEY`
- `PRODUCT`
- `EVENT`
- `CONTACT`
- `FACILITY`
- `POSITION`

Qeyd:

- `GPE` -> `LOCATION` kimi map olunur
- qalan bir çox sinif `O`-ya çevrilərək atılır

## GLiNER formatı

Notebook dataseti aşağıdakı formata çevirir:

```json
{
  "tokenized_text": ["Heyvan", "sahibinə", "külli", "miqdarda", "maddi", "ziyan", "vurulub", "."],
  "ner": [[2, 3, "quantity"]]
}
```

Bu çevirmənin nəticəsi olaraq notebook aşağıdakı faylları yarada bilir:

- `train_gliner.json`
- `val_gliner.json`
- `test_gliner.json`

## Təlim ssenariləri

Notebook-da iki əsas təlim istiqaməti görünür:

### 1. Daha böyük model ilə sınaq

- baza model: `urchade/gliner_multi-v2.1`
- `MAX_TOKENS = 96`
- `train_data_small`: ilk `10,000` nümunə
- `val_data_small`: ilk `1,000` nümunə
- `max_steps = 300`
- `batch_size = 1`

Bu sınaq zamanı MPS yaddaş çatışmazlığı xətası alınıb.

### 2. Daha kiçik model ilə sınaq

- baza model: `urchade/gliner_small-v2.1`
- qısa cümlələrdən ibarət alt seçim istifadə olunur
- `train_tiny2`: `300` nümunə
- `val_tiny2`: `30` nümunə
- `max_steps = 20`
- `batch_size = 1`
- çıxış qovluğu: `gliner_az_model_small_v2`

Bu ssenari notebook-da uğurla irəliləyən yüngül training yoxlaması kimi görünür.

## Tələblər

Layihəni işlətmək üçün ən azı bunlar lazımdır:

- Python 3
- Jupyter Notebook və ya JupyterLab
- `datasets`
- `gliner`
- `transformers`
- `torch`
- `accelerate`
- `seqeval`

Quraşdırma nümunəsi:

```bash
pip install -U datasets gliner transformers torch accelerate seqeval jupyter
```

## İstifadə

1. Repo-nu klonla.
2. Virtual environment yaratmaq tövsiyə olunur.
3. Hugging Face girişi lazımdırsa `huggingface-cli login` et.
4. Jupyter-i aç:

```bash
jupyter notebook
```

5. [GLINER.ipynb](/Users/nazrinaliyeva/Desktop/GLiNER_train-1/GLINER.ipynb)-u ardıcıllıqla işə sal.

## Qeydlər

- Notebook daxilində həm tam label seti, həm də sadələşdirilmiş label seti ilə eksperiment var.
- Böyük checkpoint faylları repo-ya daxil edilmir və `.gitignore` ilə qorunur.
- Apple Silicon `MPS` üzərində böyük model training-i yaddaş problemi verə bilər.
- Kiçik model və qısa sequence-lərlə başlamaq daha təhlükəsizdir.

## Növbəti addımlar

- training kodunu ayrıca `.py` skriptinə çıxarmaq
- evaluation metric-lərini README-də ayrıca göstərmək
- inferens nümunəsi əlavə etmək
- yaranan `train_gliner.json` və sadələşdirilmiş dataset fayllarını ayrıca versiyalamaq
