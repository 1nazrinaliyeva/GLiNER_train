# GLiNER Azerbaijani NER Training

Bu layihə Azərbaycan dili üçün NER datasetini təmizləmək, GLiNER formatına çevirmək və GLiNER modeli ilə ilkin fine-tuning təcrübəsi aparmaq üçün hazırlanıb.

Əsas iş faylı: [`GLINER.ipynb`](GLINER.ipynb)

## Məqsəd

Notebook aşağıdakı işləri görür:

- `LocalDoc/azerbaijani-ner-dataset` datasetini Hugging Face-dən yükləyir
- `tokens` və `ner_tags` sütunlarını normal Python list formatına çevirir
- pozulmuş və uzunluğu uyğun gəlməyən sətrləri filtrdən keçirir
- dataset-i `train`, `validation` və `test` hissələrinə bölür
- token-level NER annotation-larını GLiNER üçün span formatına çevirir
- label-ları analiz edir və sadələşdirilmiş label seti hazırlayır
- GLiNER modeli üzərində kiçik training sınaqları aparır

## Dataset

İstifadə olunan dataset:

```text
LocalDoc/azerbaijani-ner-dataset
```

Dataset-də hər nümunə əsasən iki sahədən ibarətdir:

- `tokens`: cümlənin token-ləri
- `ner_tags`: hər token üçün label ID

Notebook bu sahələri təmizləyir və model training-i üçün daha uyğun formata gətirir.

## Label Sadələşdirməsi

İlkin dataset-də label sayı çoxdur və bəzi label-lar bir-birinə semantik olaraq yaxındır. Buna görə notebook-da label-ların bir hissəsi saxlanılır, bəziləri isə birləşdirilir və ya `O` kimi atılır.

Saxlanılan əsas label-lar:

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
- `NUMBER`

Əlavə çevirmələr:

- `GPE` -> `LOCATION`
- `CARDINAL`, `ORDINAL`, `QUANTITY`, `PERCENTAGE` -> `NUMBER`

## GLiNER Formatı

GLiNER token-level label-lar əvəzinə span formatı gözləyir. Ona görə dataset aşağıdakı struktura çevrilir:

```json
{
  "tokenized_text": ["Ilham", "Aliyev", "Bakida", "gorushdu"],
  "ner": [
    [0, 1, "person"],
    [2, 2, "location"]
  ]
}
```

Burada:

- `tokenized_text` cümlənin token-ləridir
- `ner` entity span-larını saxlayır
- span formatı `[start_index, end_index, label]` şəklindədir

## Quraşdırma

Python virtual environment yaratmaq tövsiyə olunur:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

Lazımi paketlər:

```bash
pip install -U datasets gliner transformers torch accelerate seqeval jupyter
```

Dataset və model Hugging Face-dən yükləndiyi üçün lazım olsa giriş edin:

```bash
huggingface-cli login
```

## İstifadə

Notebook-u açın:

```bash
jupyter notebook GLINER.ipynb
```

Sonra cell-ləri ardıcıllıqla işə salın:

1. Dataset-i yüklə
2. `tokens` və `ner_tags` sahələrini təmizlə
3. Dataset-i train/validation/test hissələrinə böl
4. Label statistikalarını yoxla
5. Dataset-i GLiNER formatına çevir
6. Kiçik training sınağını işə sal

## Yaranan Fayllar

Notebook işlədikdən sonra bu fayllar yarana bilər:

- `train_gliner.json`
- `val_gliner.json`
- `test_gliner.json`
- `gliner_az_model/`
- `gliner_az_model_small_v2/`

Model checkpoint-ləri və böyük training artefaktları Git-ə əlavə edilməməlidir.

## Qeydlər

- Böyük GLiNER modeli Apple Silicon `MPS` üzərində yaddaş problemi yarada bilər.
- İlk sınaqlar üçün az nümunə, qısa cümlələr və kiçik model istifadə etmək daha rahatdır.
- Notebook-da həm ilkin label seti, həm də sadələşdirilmiş label seti ilə eksperiment var.
- Köhnə Jupyter output-ları VS Code-da notebook-un gec açılmasına səbəb ola bilər. Belə hal olsa, output-ları təmizləmək lazımdır.
