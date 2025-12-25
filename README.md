# Label Studio ile Özel YOLO Modeli Entegrasyonu

Bu kılavuz, Label Studio'yu özel bir YOLO model arka ucu (backend) ile kurma adımlarını özetlemektedir. Bu kurulum, kendi eğittiğiniz YOLO modellerini (`.pt` dosyaları) kullanarak ön etiketleme (destekli etiketleme) yapmanızı sağlar ve özellikle büyük veri setlerinin yerel olarak işlenmesi için tasarlanmıştır.

## 1. Label Studio Kurulumu ve Ayarları

### Ön Koşullar

- **Python:** Sürüm 3.10 veya üzeri
    
- **Docker:** ML Backend için gereklidir
    

### Adım 1: Label Studio'yu Yükleyin

Paketi pip kullanarak yükleyin:

Bash

```
pip install label-studio
```

### Adım 2: Yerel Dosya Sunumunu Yapılandırın

Label Studio'nun yerel diskinizde depolanan görüntülere erişebilmesi için (büyük veri setleri için gereklidir), belirli ortam değişkenlerini ayarlamanız gerekir. Aşağıdaki satırları kabuk (shell) yapılandırma dosyanıza (örneğin `.bashrc` veya `.zshrc`) ekleyin:

Bash

```
export LABEL_STUDIO_LOCAL_FILES_SERVING_ENABLED=true
export LABEL_STUDIO_LOCAL_FILES_DOCUMENT_ROOT=/home/{kullanıcı_adın}/
```

_Not: Kaydettikten sonra değişikliklerin geçerli olması için terminalinizi yeniden başlatın veya dosyayı kaynak gösterin (source ~/.bashrc)._

### Adım 3: Sunucuyu Başlatın

Label Studio'yu başlatın. Genellikle `http://localhost:8080` adresinde başlayacaktır.

Bash

```
label-studio
```

---

## 2. ML Backend (YOLO) Kurulumu
[resmi dökümantasyon linki](https://labelstud.io/guide/ml_tutorials/yolo)

### Adım 1: Repoyu Klonlayın

Label Studio ML backend deposunu indirin ve YOLO örnek dizinine gidin:

Bash

```
git clone https://github.com/HumanSignal/label-studio-ml-backend.git
cd label-studio-ml-backend/label_studio_ml/examples/yolo
```

### Adım 2: GPU Yapılandırması (İsteğe Bağlı)

Desteklenen bir GPU'nuz varsa ve bunu çıkarım (inference) için kullanmak istiyorsanız, `Dockerfile` dosyasını açın ve temel imajın (PyTorch/CUDA sürümü) yerel sürücü kurulumunuzla eşleştiğinden emin olun.

Bash

```
nano Dockerfile
```

### Adım 3: Ortam Yapılandırması

`yolo` dizininde bir `.env` dosyası oluşturun. API anahtarınızı Label Studio arayüzündeki **Account & Settings** (Hesap ve Ayarlar) sayfasından alın.

**Dosya:** `.env`

Code snippet

```
LOG_LEVEL=DEBUG
LABEL_STUDIO_API_KEY={kendi-api-key'in}
TASK_TYPE=detection 
# TASK_TYPE'ı 'detection' veya 'segmentation' olarak ayarlayın
```

### Adım 4: Docker ile Çalıştırın

Backend konteynerini oluşturun ve başlatın. Docker yüklü değilse, resmi [Docker kurulum kılavuzunu](https://docs.docker.com/engine/install/ubuntu/) izleyin.

Bash

```
docker compose up
```

---

## 3. Büyük Veri Setlerinin İçe Aktarılması (Yerel Depolama)

Büyük veri setleri için doğrudan tarayıcı yüklemesinden kaçının. Bunun yerine, yerel bir dizini senkronize edin.

1. **Proje Oluşturun:** Label Studio'da yeni bir proje oluşturun ve **Object Detection** (Nesne Tespiti) şablonunu seçin.
    
2. **İçe Aktarmayı Atlayın:** İlk veri içe aktarma adımını atlayın.
    
3. **Kaynak Ekle:**
    
    - **Settings > Cloud Storage** yolunu izleyin.
        
    - **Add Source** butonuna tıklayın.
        
    - **Local files** seçeneğini seçin.
        
4. **Yolu Yapılandırın:** Ayarları aşağıdaki görselde gösterildiği gibi doldurun. Veri setinizin **Mutlak yerel yolunu** (Absolute local path) girin (bu yol, Bölüm 1'de yapılandırılan `DOCUMENT_ROOT` içinde olmalıdır).
<img src="https://github.com/MHLeventoglu/label-studio-yolo-ml-backend-integration/blob/main/source-storage.png?raw=true" height="300px" >
5. **Senkronize Et:** Ayarları kaydedin, ardından görüntülerinizi dizine eklemek için **Sync Storage** butonuna tıklayın.
    

---

## 4. Model Entegrasyonu ve Yapılandırma

### Adım 1: Özel Modeli Dağıtın

Eğittiğiniz YOLO modelini (`.pt` dosyası) backend'in models dizinine taşıyın:

`label-studio-ml-backend/label_studio_ml/examples/yolo/models/`

### Adım 2: Label Studio'ya Bağlayın

1. Projenizde **Settings > Model** yolunu izleyin.
    
2. **Connect Model** butonuna tıklayın.
    
3. Ayrıntıları aşağıdaki görseldeki gibi doldurun. **Backend URL**'sinin Docker konteynerinizi işaret ettiğinden emin olun (genellikle `http://localhost:9090`).
<img src="https://github.com/MHLeventoglu/label-studio-yolo-ml-backend-integration/blob/main/connect-model.png?raw=true" height="300px" >

### Adım 3: Etiketleme Arayüzünü Yapılandırın

Özel model dosyanıza ve etiketlerinize başvurmak için XML yapılandırmasını güncelleyin. **Settings > Labeling Interface** yolunu izleyin.

**Önemli:**

- `model-name.pt` kısmını gerçek dosya adınızla değiştirin.
    
- `<Label>` içindeki `value` nitelikleri, YOLO modelinizin eğitildiği sınıf adlarıyla **birebir eşleşmelidir**.
    

XML

```
<View>
  <Image name="image" value="$image"/>
  
  <RectangleLabels name="label" toName="image" 
          model_path="model-name.pt" 
          model_score_threshold="0.25">
      
      <Label category="0" value="label-name-0" background="red"/>
      <Label category="1" value="label-name-1" background="green"/>
      </RectangleLabels>
</View>
```

### Adım 4: Entegrasyonu Doğrulayın

1. **Settings > Model** sekmesine dönün.
    
2. Eklediğiniz model kartını bulun ve **üç nokta** menü simgesine tıklayın.
    
3. **Send test request** seçeneğini seçin.
    
4. Tahmin işleminin başarıyla gerçekleştiğini doğrulamak için `docker compose` komutunun çalıştığı terminaldeki logları kontrol edin.
