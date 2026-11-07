# Görüntü Altyazısı Üretimi (Image Captioning)

Derin Öğrenme dersi dönem sonu projem. Flickr8k veri setiyle, bir fotoğraftan İngilizce cümle üreten bir model kurdum. Model InceptionV3 ile görüntü özelliği çıkarıyor, bir encoder ile küçültüyor, ardından Bahdanau attention ve GRU decoder ile kelime kelime altyazı üretiyor. Kod TensorFlow/Keras ile yazıldı ve Google Colab'da (GPU) çalıştırıldı.

## Veri seti

[Flickr8k](https://github.com/jbrownlee/Datasets/releases/tag/Flickr8k): 8.092 fotoğraf, her biri için 5 altyazı (toplam 40.460). Notebook klasörde bulabildiği 8.091 görüntü için özellik çıkardı. Veri bu repoda yok, notebook'un ikinci hücresi `wget` ile indiriyor.

Altyazıları küçük harfe çevirdim, noktalama işaretlerini sildim ve başlarına `startseq`, sonlarına `endseq` ekledim. Sözlükte 8.781 farklı kelime çıktı, modelde en sık geçen 5.000 tanesini kullandım, gerisi `<unk>` oldu. Cümleleri 50 uzunluğa tamamladım (padding).

## Model

1. **InceptionV3** (ImageNet ağırlıkları): sınıflandırma katmanından önceki çıktı alınıyor, her görüntü için 2048 boyutlu bir vektör. Bu özellikler bir kez çıkarılıp saklanıyor, InceptionV3 eğitilmiyor.
2. **CNN Encoder**: Dense(256) + ReLU, 2048 boyutlu vektörü 256'ya indiriyor.
3. **Bahdanau Attention**
4. **GRU Decoder**: embedding boyutu 256, GRU 512 birim, sonda kelime dağarcığı boyutunda (5.000) Dense katman.

| Parametre | Değer |
|-----------|-------|
| Batch size | 64 |
| Optimizer | Adam (varsayılan ayarlar) |
| Kayıp | Maskelenmiş Sparse Categorical Cross-Entropy (padding hesaba katılmıyor) |
| Eğitim yöntemi | Teacher forcing |
| Epoch (kodda) | 50 |
| Epoch (yapılabilen) | 5 |

## Eğitim sonuçları

Kodda 50 epoch yazıyor ama Colab GPU süresi yetmediği için 5 epoch eğitebildim. Bir epoch yaklaşık 21 dakika sürdü, toplam yaklaşık 107 dakika.

| Epoch | Ortalama eğitim kaybı | Süre (sn) |
|-------|----------------------|-----------|
| 1 | 0.9782 | 1273.0 |
| 2 | 0.8328 | 1288.7 |
| 3 | 0.7918 | 1294.4 |
| 4 | 0.7700 | 1289.0 |
| 5 | 0.7539 | 1277.1 |

Kayıp 0.9782'den 0.7539'a indi.

![Eğitim kaybı](results/loss_curve.png)

E�itimden önce veri setinden rastgele seçtiğim görüntüler ve temizlenmiş altyazıları:

![Örnek görüntüler](results/sample_images.png)

## Eksikler

- Sadece eğitim kaybına baktım. Doğrulama seti ve BLEU gibi bir kalite metriği yok, o yüzden altyazıların ne kadar iyi olduğunu sayıyla söyleyemiyorum.
- Notebook'ta modelin ürettiği örnek altyazılar gösterilmiyor, yani çıkarım (inference) kodu yok.
- Attention, görüntü bölgelerine değil, 256 boyutlu tek bir vektöre uygulanıyor. Bu yüzden uzamsal attention (modelin görüntünün neresine baktığı) yok.
- Eğitim hücresi `dataset` ve `X_img` değişkenlerini kullanıyor, ama bunları oluşturan hücre notebook'ta yok. Notebook bu haliyle baştan sona tekrar çalıştırılamaz.

## Klasörler

- `notebooks/image_captioning.ipynb`: kodun kendisi
- `reports/derin_ogrenme_raporu.pdf`: dönem sonu raporu
- `results/`: grafikler

## Çalıştırma

Colab'da GPU açıp notebook'u yüklemek yeterli, veriyi kendisi indiriyor. Kendi bilgisayarında çalıştırmak istersen:

```bash
pip install -r requirements.txt
jupyter notebook notebooks/image_captioning.ipynb
```

Taha Doğukan Bahadır
