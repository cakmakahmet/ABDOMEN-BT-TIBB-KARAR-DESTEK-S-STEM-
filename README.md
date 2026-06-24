# Abdomen BT Analiz: RT-DETR/RF-DETR Tabanlı Acil Patoloji Ön Değerlendirme Sistemi

Bu proje, abdomen bilgisayarlı tomografi (BT) görüntülerinde acil değerlendirme gerektirebilecek patolojik bulguların yapay zekâ destekli olarak ön değerlendirilmesi amacıyla geliştirilmiştir. Sistem; derin öğrenme tabanlı nesne tespiti, açıklanabilir yapay zekâ (XAI), RAG/LLM destekli raporlama ve web tabanlı kullanıcı arayüzü bileşenlerini bir araya getiren uçtan uca bir klinik karar destek prototipidir.

Proje kapsamında abdomen BT görüntülerinde patolojik bölgelerin sınıfı ve konumu nesne tespiti yaklaşımıyla belirlenmekte, model çıktıları bounding box, confidence skoru, XAI ısı haritaları ve LLM destekli açıklayıcı raporlar ile kullanıcıya sunulmaktadır. Sistem kesin tanı koymak amacıyla değil, radyolojik değerlendirme sürecini destekleyebilecek bir ön değerlendirme aracı olarak tasarlanmıştır.

## Projenin Amacı

Acil servis ve radyoloji iş akışlarında abdomen BT görüntüleri; pankreatit, kolesistit, üriner sistem taşı, apandisit/divertikül ilişkili inflamasyon ve aort patolojileri gibi hızlı değerlendirme gerektirebilecek durumlar için sık kullanılan görüntüleme yöntemlerindendir. Ancak BT incelemeleri çok sayıda kesit içerdiğinden, yoğun klinik ortamda dikkat gerektiren bölgelerin hızlı şekilde belirlenmesi zorlaşabilir.

Bu projenin amacı, tek bir abdomen BT kesiti üzerinde olası patolojik bölgenin sınıfını ve konumunu belirleyebilen, model kararlarını açıklanabilir yapay zekâ yöntemleriyle görselleştiren ve RAG/LLM desteğiyle güvenli bir ön değerlendirme raporu üretebilen bir karar destek sistemi geliştirmektir.

## Tespit Edilen Patoloji Sınıfları

Proje beş sınıflı bir nesne tespiti problemi olarak ele alınmıştır:

* Aort hastalığı
* Pankreatit
* Kolesistit
* Taş hastalığı
* Apandisit ve divertikül patolojisi

## Kullanılan Veri Seti

Projede abdomen BT görüntülerinden oluşan etiketli bir veri seti kullanılmıştır. Veri seti eğitim ve doğrulama ayrımıyla düzenlenmiş, görüntülere ait patolojik bölgeler bounding box anotasyonlarıyla işaretlenmiştir.

Veri seti yapısı:

* Eğitim görüntüsü: 15143
* Eğitim kutu etiketi: 16760
* Doğrulama görüntüsü: 4640
* Doğrulama kutu etiketi: 4971

Sınıf dağılımında belirgin dengesizlikler bulunmaktadır. Özellikle apandisit/divertikül sınıfının doğrulama etiketlerinde temsil edilmemesi, bu sınıfa ait performans yorumlarını sınırlamaktadır. Taş hastalığı sınıfı ise küçük nesne yapısı nedeniyle lokalizasyon açısından daha zorlayıcı bir sınıf olarak değerlendirilmiştir.

## Veri Ön İşleme

PNG ve JPEG formatındaki görüntüler doğrudan model çıkarımına verilebilirken, DICOM görüntüler için eğitim süreciyle uyumlu özel bir ön işleme akışı uygulanmıştır.

DICOM görüntülerde:

* Piksel değerleri RescaleSlope ve RescaleIntercept kullanılarak Hounsfield Unit değerlerine çevrilmiştir.
* MONOCHROME1 görüntüler gerekli durumda terslenmiştir.
* Görüntüler 512x512 boyutuna getirilmiştir.
* Üç farklı pencereleme sonucu pseudo-RGB formatında temsil edilmiştir.

Kullanılan pseudo-RGB kanal yapısı:

| Kanal | Pencere                | Amaç                                          |
| ----- | ---------------------- | --------------------------------------------- |
| R     | Abdomen penceresi      | Genel abdominal doku görünürlüğü              |
| G     | Kemik/Taş penceresi    | Taş ve kalsifiye yapı kontrastı               |
| B     | Yumuşak doku penceresi | Organ sınırları ve inflamatuvar değişiklikler |

Bu yaklaşım, tek kanallı BT kesitindeki farklı doku yoğunluğu aralıklarını modele aynı anda sunmayı amaçlamaktadır. Özellikle küçük ve yüksek yoğunluklu taş bulgularının görünürlüğünü artırmak için önemlidir.

## Kullanılan Modeller

Proje kapsamında farklı nesne tespiti yaklaşımları denenmiş ve sonuçları karşılaştırılmıştır.

Deneylerde kullanılan başlıca modeller:

* RT-DETR / RF-DETR
* YOLOv11 ailesi
* YOLOv11 pseudo-RGB
* YOLOv11 CLAHE
* ResDet tabanlı denemeler
* ResDet50 P2 CLAHE
* YOLO P2 self-attention
* Faster R-CNN denemeleri

Deneysel sonuçlarda en güçlü performans RT-DETR Abdomen koşusunda elde edilmiştir. Bu nedenle backend mimarisi RT-DETR/RF-DETR birincil model olacak şekilde tasarlanmıştır.

## Model Performansları

Farklı model denemeleri precision, recall, mAP@50 ve mAP@50-95 metrikleriyle değerlendirilmiştir.

| Model                  | Precision | Recall | mAP@50 | mAP@50-95 |
| ---------------------- | --------: | -----: | -----: | --------: |
| RT-DETR Abdomen        |     0.783 |  0.747 |  0.745 |     0.421 |
| YOLOv11m pseudo-RGB    |     0.742 |  0.707 |  0.727 |     0.404 |
| YOLOv11l CLAHE A100    |     0.740 |  0.667 |  0.702 |     0.396 |
| ResDet50 P2 CLAHE      |     0.760 |  0.699 |  0.727 |     0.392 |
| ResDet50 fine-tuning   |     0.744 |  0.705 |  0.717 |     0.390 |
| YOLOv11l final         |     0.745 |  0.711 |  0.699 |     0.389 |
| YOLO P2 self-attention |     0.779 |  0.707 |  0.664 |     0.369 |
| YOLOv11l 768 no mosaic |     0.722 |  0.683 |  0.687 |     0.359 |

En iyi deneysel model RT-DETR olarak seçilmiştir. Bu model, mAP@50-95, mAP@50, precision ve recall değerleri birlikte değerlendirildiğinde diğer modellere göre daha dengeli bir performans göstermiştir.

## Sistem Mimarisi

Geliştirilen sistem web tabanlı bir karar destek prototipi olarak tasarlanmıştır. Kullanıcı bir abdomen BT görüntüsü yükler, backend görüntüyü işler, model çıkarımı yapılır, tespit sonuçları görselleştirilir, XAI çıktıları üretilir ve RAG/LLM destekli rapor oluşturulur.

Sistemin ana bileşenleri:

| Modül         | Teknoloji                      | Görev                                                   |
| ------------- | ------------------------------ | ------------------------------------------------------- |
| Frontend      | React + TypeScript             | Görüntü yükleme, analiz, XAI, rapor ve export ekranları |
| Backend API   | FastAPI                        | REST servisleri, dosya doğrulama, analiz orkestrasyonu  |
| Model Servisi | RT-DETR/RF-DETR, YOLO fallback | Patoloji sınıfı ve bounding box üretimi                 |
| DICOM İşleme  | pydicom + OpenCV               | HU dönüşümü, pencereleme, metadata çıkarımı             |
| XAI           | EigenCAM + Grad-CAM            | Model dikkat bölgelerinin görselleştirilmesi            |
| RAG/LLM       | FAISS + embeddings + LLM       | Klinik bağlam getirme ve rapor üretimi                  |
| Export        | JSON / Markdown / ZIP          | Vaka çıktılarının dışa aktarımı                         |

## Backend Yapısı

Backend FastAPI ile geliştirilmiştir. Temel servisler:

* Dosya yükleme
* Analiz çalıştırma
* Model bilgisi sorgulama
* Sistem durumu kontrolü
* XAI üretimi
* RAG bağlam getirme
* Markdown / JSON / ZIP export

Model servisinde sistem önce `models/rtdetr_best.pt` ağırlığını arar. Bu dosya mevcutsa RT-DETR/RF-DETR tabanlı çıkarım yapılır. Dosya bulunmadığında sistem çalışabilirliği korumak amacıyla `models/best.pt` YOLO uyumluluk ağırlığına fallback yapar. Bu durum sistem durumu ve model bilgisi çıktılarında açıkça raporlanır.

## Frontend ve Kullanıcı Akışı

Frontend React, TypeScript ve Vite kullanılarak geliştirilmiştir. Kullanıcı arayüzü vaka odaklı bir analiz akışı sunar.

Genel kullanıcı akışı:

1. Kullanıcı PNG, JPEG veya DICOM formatında abdomen BT görüntüsü yükler.
2. Confidence ve dil seçenekleri belirlenir.
3. Backend görüntüyü doğrular ve gerekirse DICOM ön işleme uygular.
4. RT-DETR/RF-DETR tabanlı model çıkarımı yapılır.
5. Tespit edilen patoloji sınıfı, confidence değeri ve bounding box gösterilir.
6. XAI sekmesinden EigenCAM veya Grad-CAM çıktısı üretilebilir.
7. RAG/LLM destekli Türkçe veya İngilizce ön değerlendirme raporu oluşturulur.
8. Vaka çıktıları ZIP olarak dışa aktarılabilir.

## Açıklanabilir Yapay Zekâ

Tıbbi görüntü analizi sistemlerinde modelin yalnızca tahmin üretmesi yeterli değildir. Modelin görüntünün hangi bölgelerine odaklanarak karar verdiğinin de incelenmesi gerekir. Bu nedenle projede açıklanabilir yapay zekâ yöntemleri kullanılmıştır.

Kullanılan XAI yöntemleri:

* EigenCAM
* Grad-CAM

EigenCAM, modelin özellik haritalarındaki baskın aktivasyon bölgelerini gradyan gerektirmeden görselleştirir. Grad-CAM ise hedef sınıfa göre gradyan bilgisini kullanarak sınıfa özgü önem bölgelerini gösterir.

XAI çıktıları model kararını yorumlamaya yardımcı görsellerdir. Ancak bu görseller tek başına modelin doğru karar verdiğini kanıtlamaz. Isı haritaları, uzman değerlendirmesini destekleyen yardımcı çıktılar olarak ele alınmalıdır.

## RAG/LLM Destekli Raporlama

Proje kapsamında model çıktılarının daha anlaşılır hale getirilmesi için RAG/LLM destekli raporlama modülü geliştirilmiştir. Bilgi tabanı, `knowledge_base` klasöründeki Markdown dosyalarından oluşturulmuştur. FAISS tabanlı vektör veritabanı aracılığıyla ilgili patolojiye ait klinik bağlam getirilmekte ve LLM bu bağlamı kullanarak güvenli bir ön değerlendirme raporu üretmektedir.

Raporlarda:

* Modelin tespit ettiği sınıf
* Confidence değeri
* Bounding box bilgisi
* DICOM ön işleme ayrıntıları
* İlgili klinik bağlam
* Güvenli karar destek açıklaması

yer almaktadır.

LLM raporları kesin tanı veya tedavi önerisi üretmek için kullanılmaz. Rapor çıktıları yalnızca ön değerlendirme ve karar destek amacı taşır.

## Değerlendirme Metrikleri

Model performansları aşağıdaki metrikler kullanılarak değerlendirilmiştir:

* Precision
* Recall
* mAP@50
* mAP@50-95
* Confusion matrix
* Precision-recall eğrisi
* F1-confidence eğrisi
* Eğitim ve doğrulama loss grafikleri

mAP@50 daha gevşek IoU eşiğinde tespit başarısını gösterirken, mAP@50-95 farklı IoU eşiklerinin ortalamasını aldığı için lokalizasyon kalitesini daha sıkı değerlendirir. Bu nedenle model seçiminde yalnızca precision veya mAP@50 değil, mAP@50-95 ve recall dengesi birlikte dikkate alınmıştır.

## Proje Çıktıları

Bu proje kapsamında elde edilen başlıca çıktılar:

* Abdomen BT görüntülerinde patoloji tespiti yapan derin öğrenme modelleri
* RT-DETR/RF-DETR birincil çıkarım servisi
* YOLO uyumluluk fallback mekanizması
* DICOM görüntüler için HU dönüşümü ve pseudo-RGB pencereleme
* Bounding box ve confidence skoru tabanlı tespit çıktıları
* EigenCAM ve Grad-CAM tabanlı XAI görselleri
* RAG/LLM destekli Türkçe ve İngilizce ön değerlendirme raporları
* React tabanlı kullanıcı arayüzü
* FastAPI tabanlı backend
* JSON, Markdown ve ZIP export desteği

## Örnek Proje Yapısı

```text
abdomenbt-xai-rag/
│
├── backend/
│   ├── main.py
│   ├── detector_service.py
│   ├── dicom_utils.py
│   ├── xai_service.py
│   └── rag_service.py
│
├── frontend/
│   ├── src/
│   ├── public/
│   └── package.json
│
├── data/
│   ├── train/
│   └── val/
│
├── models/
│   ├── rtdetr_best.pt
│   └── best.pt
│
├── knowledge_base/
│   ├── aort.md
│   ├── pankreatit.md
│   ├── kolesistit.md
│   ├── tas.md
│   └── apandisit_divertikul.md
│
├── outputs/
│   ├── cases/
│   ├── xai/
│   └── exports/
│
├── results/
│   ├── confusion_matrix.png
│   ├── pr_curve.png
│   ├── f1_curve.png
│   └── training_results.png
│
├── README.md
└── requirements.txt
```

## Kurulum

Backend için gerekli Python kütüphanelerini yüklemek:

```bash
pip install -r requirements.txt
```

Backend çalıştırmak:

```bash
uvicorn backend.main:app --reload
```

Frontend klasörüne geçip bağımlılıkları yüklemek:

```bash
cd frontend
npm install
```

Frontend çalıştırmak:

```bash
npm run dev
```

## Kullanım

1. Uygulama arayüzünden abdomen BT görüntüsü yüklenir.
2. Confidence eşiği ve rapor dili seçilir.
3. Analiz başlatılır.
4. Model tarafından tespit edilen sınıf, confidence skoru ve bounding box görüntülenir.
5. XAI sekmesinden EigenCAM veya Grad-CAM çıktıları üretilir.
6. RAG/LLM destekli rapor incelenir.
7. İstenirse vaka çıktıları ZIP olarak dışa aktarılır.

## Sınırlılıklar

Bu sistem tek kesit veya yüklenen tek görüntü üzerinden çalışacak şekilde tasarlanmıştır. Tüm BT serisinin 3B bağlamını değerlendirmez. Bu durum özellikle küçük taşların kesitler arası takibi, inflamasyon yayılımı ve vasküler yapıların hacimsel analizi açısından sınırlılık oluşturur.

Ayrıca veri setinde sınıf dengesizliği bulunmaktadır. Özellikle apandisit/divertikül sınıfının doğrulama etiketlerinde temsil edilmemesi, bu sınıf için performans yorumunu sınırlandırmaktadır.

LLM raporları model çıktısına ve bilgi tabanına bağlıdır. Klinik öykü, laboratuvar değerleri, vital bulgular ve uzman radyolog yorumu olmadan kesin tanı veya tedavi planı oluşturamaz.

## Gelecek Çalışmalar

Gelecekte yapılabilecek geliştirmeler:

* Daha dengeli ve geniş doğrulama/test seti oluşturulması
* Tüm BT serisini değerlendirebilen 3B model desteği
* RT-DETR’ye özel XAI hedef katman adaptörünün tamamlanması
* DICOM seri düzeyinde analiz ve export desteği
* Aktif öğrenme ile uzman etiket düzeltme döngüsü
* Çok merkezli veri ile dış doğrulama
* Radyolog değerlendirmesiyle karşılaştırmalı klinik çalışma
* RAG/LLM tarafında kaynak gösterimli raporlama
* Daha sıkı halüsinasyon kontrolü ve güvenlik filtresi

## Önemli Not

Bu proje akademik ve araştırma amaçlı geliştirilmiş bir karar destek prototipidir. Sistem tarafından üretilen tespitler, XAI görselleri ve LLM raporları kesin tanı niteliği taşımaz. Nihai klinik karar, uzman hekim tarafından hastanın klinik bulguları, laboratuvar sonuçları ve gerekli ek tetkikleri ile birlikte verilmelidir.

## Kaynaklar

* T.C. Sağlık Bakanlığı Açık Veri Portalı, TR_Abdomen Rad_Emergency Dataset
  https://acikveri.saglik.gov.tr/Home/DataSets?categoryId=11

* Ultralytics Documentation: YOLO ve RT-DETR model eğitimi, çıkarımı ve metrik raporlama.

* Selvaraju, R. R. ve ark. Grad-CAM: Visual Explanations from Deep Networks via Gradient-based Localization.

* Muhammad, M. B. ve Yeasin, M. Eigen-CAM: Class Activation Map using Principal Components.

* DICOM Standard: Digital Imaging and Communications in Medicine.

* FAISS ve LangChain dokümantasyonları: Vektör tabanlı bilgi getirme ve RAG uygulamaları.
