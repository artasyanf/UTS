
# UTS Graf Sains Data: Neo4j dan Rekomendasi

## Pembagian Fold

| Fold          | Periode                 |
| ------------- | ----------------------- |
| Training (P1) | 24/05/2015 - 24/07/2015 |
| Testing (P2)  | 02/08/2015 - 17/08/2015 |

Timestamp pada data berformat milidetik (UTC).

## Isi Repositori

- `uts_gsd.ipynb`: seluruh pengerjaan (bagian 1.1 sampai 2.2) beserta penjelasan
- `requirements.txt`: dependensi Python
- `data/events_filtered.csv`: hasil filter fold (train + test, kolom `fold`)
- `data/events_train.csv`, `data/events_test.csv`: hasil filter per fold
- `data/item_similarity_topk.csv`: 10 tetangga terdekat per item (cosine)
- `screenshots/`: bukti graf Neo4j (kondisi sebelum dan sesudah similarity)

## Dataset

RetailRocket E-commerce Dataset (Roman Zykov), Kaggle:
https://www.kaggle.com/datasets/retailrocket/ecommerce-dataset

File mentah (`events.csv`, `item_properties_*.csv`, `category_tree.csv`)
tidak diunggah karena ukurannya besar.

## Ringkasan Metode

1. Data `events.csv` difilter menurut fold, lalu fold training diimpor ke Neo4j
   (simpul `Visitor` dan `Item`; relasi `VIEWED`, `ADDED_TO_CART`, `BOUGHT`).
2. VSM User dan VSM Item dibangun dari bobot interaksi (view=1, addtocart=3,
   transaction=5). Proyeksi **Item-Item** dipilih karena lebih efisien dan
   vektornya lebih kaya sinyal.
3. Similarity memakai cosine pada VSM Item; top-10 tetangga per item ditulis ke
   graf sebagai relasi `SIMILAR_TO`.
4. Rekomendasi memakai item-based KNN (k=5). Evaluasi dengan Hit@5 dan NDCG@5
   terhadap fold testing.

## Hasil Ringkas

**User acak (seed 42): ID 132066**

- 3 item yang diiklankan: 265571, 334041, 386518
- Hit@5 = 0 dan NDCG@5 = 0 (item relevan 465211 tidak masuk top-5)

**Rata-rata 500 user acak (seed 42)**

| Metode               | Hit@5  | NDCG@5 |
| -------------------- | ------ | ------ |
| Baseline popularitas | 0,0080 | 0,0023 |
| Item-KNN (k=5)       | 0,0180 | 0,0057 |

Nilai absolut rendah karena data sangat jarang (80,5% user hanya berinteraksi
dengan 1 item) dan fold testing hanya 16 hari.

## Cara Menjalankan

1. `uv venv myvenv --python 3.10` lalu `source myvenv/bin/activate`
2. `uv pip install -r requirements.txt`
3. Jalankan Neo4j versi 5.x, lalu ubah kredensial (`URI`, `AUTH`) di notebook
   sesuai instance Anda
4. Letakkan `events.csv` di folder `data/`, lalu jalankan notebook dari atas
