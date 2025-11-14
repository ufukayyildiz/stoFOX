# STOFOX: Cloudflare R2 için Geliştiricilerin Yeni Gözdesi 🦊

## Neden Her Geliştiricinin Araç Kutusunda Cloudflare R2 İstemcisi Olmalı?

2025'te bulut depolama artık lüks değil, zorunluluk. Ancak doğru aracı bulmak? İşte asıl zorluk orada. AWS S3'ün yüksek maliyetlerinden sıkıldıysanız ve Cloudflare R2'nin uygun fiyatlı depolamasına geçiş yapmak istiyorsanız, bir sorunla karşılaşırsınız: **R2'nin resmi bir masaüstü istemcisi yok**.

Terminal komutlarıyla uğraşmak mı? Rclone config dosyalarını düzenlemek mi? Her seferinde AWS CLI syntax'ını mı hatırlamaya çalışıyorsunuz? Bunların hepsi yorucu.

İşte bu yüzden **STOFOX**'u yarattık.

---

## 📦 STOFOX Nedir?

**STOFOX**, Cloudflare R2 object storage için özel olarak tasarlanmış, modern ve kullanıcı dostu bir masaüstü uygulamasıdır. Java ile yazılmış, macOS için PKG installer ile dağıtılan ve gömülü JRE sayesinde hiçbir kurulum gerektirmeyen bir çözüm.

### Temel Özellikler:

✅ **Sıfır Bağımlılık**: Embedded JRE ile tek tıkla kurulum  
✅ **Explorer-Style Arayüz**: Windows Explorer veya macOS Finder gibi tanıdık deneyim  
✅ **Hızlı Dosya İşlemleri**: Upload, download, delete, preview - hepsi birkaç tıkla  
✅ **Folder Upload**: Klasör hiyerarşilerini koruyarak toplu yükleme  
✅ **Auto-Update**: Yeni sürümler otomatik olarak kontrol edilir ve kurulur  
✅ **Analytics**: Mixpanel entegrasyonu ile kullanım metrikleri ve hata takibi  
✅ **Modern UI**: FlatLaf IntelliJ teması ile profesyonel görünüm

---

## 🎯 Problem: R2'nin Eksik Halkası

Cloudflare R2, AWS S3'e harika bir alternatif sunuyor:
- **Zero egress fees** (çıkış ücreti yok!)
- **S3-compatible API** (mevcut araçlarla çalışır)
- **Küresel dağıtım** (Cloudflare'in edge network'ü)

Ancak bir sorunu var: **Kullanıcı dostu bir masaüstü arayüzü yok**.

### Mevcut Çözümler ve Sorunları:

**1. Cloudflare Dashboard**
- ❌ Web tabanlı (yavaş, sınırlı)
- ❌ Toplu işlemler zor
- ❌ Klasör yükleme desteği yok

**2. AWS CLI / Rclone**
- ❌ Terminal bilgisi gerekiyor
- ❌ Karmaşık config dosyaları
- ❌ Hata ayıklama zor

**3. Cyberduck / CloudBerry**
- ❌ R2'ye özel optimize edilmemiş
- ❌ Gereksiz S3 özellikleri
- ❌ Ücretli sürümler

### STOFOX'un Çözümü:

✅ **Native Desktop App**: macOS için optimize edilmiş  
✅ **R2-First Design**: Sadece ihtiyacınız olan özellikler  
✅ **Açık Kaynak**: Topluluk katkılarına açık  
✅ **Ücretsiz**: Sıfır maliyet, sıfır reklam

---

## 🏗️ Mimari: Nasıl Çalışıyor?

STOFOX, modern yazılım geliştirme prensipleriyle inşa edilmiş:

### 1. **Tech Stack**

```
┌─────────────────────────────────────┐
│     Java 17 (Temurin JRE)          │
├─────────────────────────────────────┤
│  AWS SDK for Java 2.20.0 (S3 API) │
├─────────────────────────────────────┤
│    FlatLaf UI Framework 3.5.2      │
├─────────────────────────────────────┤
│     Mixpanel Analytics 1.5.3       │
└─────────────────────────────────────┘
```

**Neden Java?**
- ✅ Cross-platform uyumluluk
- ✅ Olgun S3 SDK ekosistemi
- ✅ Memory-safe ve güvenli
- ✅ JRE embedding desteği (jpackage)

**Neden AWS SDK?**
- ✅ R2'nin S3-compatible API'si
- ✅ Battle-tested kod tabanı
- ✅ Async/sync operasyon desteği
- ✅ Otomatik retry mekanizması

### 2. **UI/UX Felsefesi**

STOFOX'un arayüzü **"tanıdık ama modern"** prensibiyle tasarlandı:

**Hierarchical Navigation:**
```
My Bucket / photos / 2025 / january / 
[← Clickable breadcrumbs ile kolay navigasyon]
```

**Dual-Pane Layout:**
```
┌─────────────┬───────────────────────────┐
│  SIDEBAR    │    MAIN PANEL             │
│             │                           │
│ • Settings  │  ┌─────────────────────┐  │
│ • Refresh   │  │ Name       │  Size  │  │
│ • Check     │  ├─────────────────────┤  │
│   Updates   │  │ 📁 backups │ -      │  │
│ • About     │  │ 📄 data.json│ 2.4KB │  │
│             │  └─────────────────────┘  │
│ BUCKETS:    │                           │
│ • cdn0      │  [Upload] [Download] [...] │
│ • food360   │                           │
│ • library   │                           │
│             │                           │
│ v1.0.21     │                           │
└─────────────┴───────────────────────────┘
```

**Context Menus:**
- Right-click → Download / Delete / View
- Multi-select support (Cmd+A on macOS)
- Keyboard shortcuts

### 3. **Core Operations**

**Upload Flow:**
```java
1. User selects file/folder
2. Background thread starts
3. Progress dialog shows:
   - Overall: [████████░░] 80% (400MB / 500MB)
   - Files: 8 / 10 uploaded
4. S3 SDK uploads to R2
5. Table refreshes automatically
```

**Folder Upload:**
```java
1. Scan directory tree (with symlink protection)
2. Preserve folder structure
3. Upload files with proper S3 keys:
   - /folder/subfolder/file.txt
4. Progress tracking per file
```

**Auto-Update System:**
```java
1. Check R2 bucket for latest.json
2. Compare versions (semver)
3. If newer:
   - PKG mode: Download installer to ~/Downloads
   - JAR mode: Replace JAR, restart app
4. Show progress with byte tracking
```

---

## 🚀 Özellikler: Detaylı İnceleme

### 1. **Bucket Management**

**Otomatik Bucket Discovery:**
```java
// R2 credentials ile bağlan
R2Config config = R2Config.loadFromFile();
S3Client client = S3Client.builder()
    .endpointOverride(config.getEndpoint())
    .region(Region.of("auto"))
    .credentialsProvider(...)
    .build();

// Tüm bucket'ları listele
ListBucketsResponse response = client.listBuckets();
```

**Sidebar Navigation:**
- Bucket listesi otomatik güncellenir
- Bucket seçimi → içerik yüklenir
- Hızlı bucket değiştirme

### 2. **Object Operations**

**Upload:**
```java
// Tek dosya
PutObjectRequest request = PutObjectRequest.builder()
    .bucket(bucketName)
    .key(objectKey)
    .contentType(detectMimeType(file))
    .build();

// Progress tracking
RequestBody body = RequestBody.fromFile(file);
client.putObject(request, body);
```

**Download:**
```java
// Multi-file download
for (S3Object obj : selectedObjects) {
    GetObjectRequest request = GetObjectRequest.builder()
        .bucket(bucketName)
        .key(obj.key())
        .build();
    
    client.getObject(request, 
        ResponseTransformer.toFile(localPath));
}
```

**Delete:**
```java
// 2-step confirmation
1. Show dialog: "Delete 5 files?"
2. Red button: "Confirm Delete"
3. Background deletion with retry
```

### 3. **File Preview**

**Supported Types:**

**Images:**
- JPG, PNG, GIF, WebP
- In-app preview with scaling
- Full-screen mode

**Text:**
- TXT, JSON, XML, MD
- Syntax highlighting (future)
- UTF-8 support

**PDF:**
- Apache PDFBox integration
- First page preview
- File info (pages, size)

**Code:**
```java
// Preview window
JDialog previewDialog = new JDialog();
JTextArea textArea = new JTextArea(content);
textArea.setEditable(false);
textArea.setFont(new Font("Monospaced", Font.PLAIN, 12));
```

### 4. **Progress Tracking**

**Dual Progress Bar System:**

```
┌────────────────────────────────────┐
│  Uploading Files...                │
├────────────────────────────────────┤
│                                    │
│  Overall Progress:                 │
│  [████████████████░░] 80%          │
│  400 MB / 500 MB                   │
│                                    │
│  Files: 8 / 10 uploaded            │
│                                    │
│  Current: large-video.mp4          │
│  [████████████░░░░░░] 60%          │
│                                    │
│            [Cancel]                │
└────────────────────────────────────┘
```

**Technical Implementation:**
```java
// Byte-based progress
long totalBytes = files.stream()
    .mapToLong(File::length)
    .sum();

long uploadedBytes = 0;
for (File file : files) {
    // Upload with progress callback
    uploadedBytes += file.length();
    int percent = (int)((uploadedBytes * 100) / totalBytes);
    progressBar.setValue(percent);
}
```

### 5. **Auto-Update System**

**Version Check:**
```java
// latest.json on R2
{
  "version": "1.0.22",
  "downloadUrl": "https://r2.../STOFOX-1.0.22.pkg",
  "releaseNotes": "Bug fixes and improvements"
}

// Comparison
if (isNewerVersion("1.0.22", currentVersion)) {
    showUpdateDialog();
}
```

**PKG Mode (macOS):**
```java
1. Download PKG to ~/Downloads
2. Show notification
3. Open Finder to Downloads folder
4. User double-clicks installer
5. macOS handles installation
```

**JAR Mode (Development):**
```java
1. Download new JAR
2. Replace current JAR
3. Restart application
4. Continue seamlessly
```

### 6. **Analytics & Error Tracking**

**Mixpanel Integration:**
```java
// App launch
trackEvent("App Launched", {
    "$os": "Mac OS X",
    "$app_version": "1.0.21",
    "os_version": "14.3",
    "os_arch": "aarch64",
    "java_version": "17.0.9"
});

// Geolocation (automatic via IP)
// Country: Turkey
// City: Istanbul
```

**Error Tracking:**
```java
// Uncaught exception handler
Thread.setDefaultUncaughtExceptionHandler((t, e) -> {
    trackEvent("App Error", {
        "error_message": e.getMessage(),
        "error_type": e.getClass().getName(),
        "$os": System.getProperty("os.name"),
        "$app_version": getCurrentVersion()
    });
});
```

**Benefits:**
- ✅ Crash reporting
- ✅ User behavior insights
- ✅ Performance metrics
- ✅ Geographic distribution

---

## 📊 Performance & Optimization

### 1. **Memory Management**

**Efficient File Handling:**
```java
// Streaming upload (large files)
RequestBody.fromInputStream(
    new BufferedInputStream(new FileInputStream(file)),
    file.length()
);

// No full file load in memory
```

**UI Threading:**
```java
// Background operations
SwingWorker<Result, Progress> worker = new SwingWorker<>() {
    @Override
    protected Result doInBackground() {
        // Heavy S3 operations
        return uploadFiles();
    }
    
    @Override
    protected void done() {
        // Update UI on EDT
        refreshTable();
    }
};
```

### 2. **Network Optimization**

**AWS SDK Best Practices:**
```java
S3Client client = S3Client.builder()
    .region(Region.of("auto"))
    .endpointOverride(r2Endpoint)
    .credentialsProvider(StaticCredentialsProvider.create(
        AwsBasicCredentials.create(accessKey, secretKey)
    ))
    .httpClientBuilder(NettyNioAsyncHttpClient.builder()
        .maxConcurrency(50)
        .connectionTimeout(Duration.ofSeconds(30))
    )
    .build();
```

**Retry Logic:**
```java
RetryPolicy retryPolicy = RetryPolicy.builder()
    .numRetries(3)
    .backoffStrategy(BackoffStrategy.exponentialDelay())
    .build();
```

### 3. **Caching**

**Object List Caching:**
```java
// Cache bucket contents
Map<String, List<S3Object>> bucketCache = new HashMap<>();

// Invalidate on:
// - Manual refresh
// - After upload/delete
// - Auto-refresh timer
```

---

## 🔐 Güvenlik

### 1. **Credential Management**

**Secure Storage:**
```java
// Config file: r2client-config.json
{
    "accountId": "abc123",
    "accessKeyId": "R2_ACCESS_KEY",
    "secretAccessKey": "R2_SECRET_KEY",
    "selectedBucket": "my-bucket"
}

// Permissions: 600 (read/write owner only)
```

**Best Practices:**
- ✅ No hardcoded credentials
- ✅ Platform-specific storage paths
- ✅ No logging of secrets
- ✅ Secure transmission (HTTPS)

### 2. **Error Handling**

**Graceful Degradation:**
```java
try {
    uploadFile(file);
} catch (S3Exception e) {
    if (e.statusCode() == 403) {
        showError("Access denied. Check credentials.");
    } else if (e.statusCode() == 404) {
        showError("Bucket not found.");
    } else {
        showError("Upload failed: " + e.getMessage());
    }
}
```

### 3. **Input Validation**

**Sanitization:**
```java
// Bucket name validation
Pattern bucketPattern = Pattern.compile("^[a-z0-9][a-z0-9-]*[a-z0-9]$");

// Object key validation
String sanitizedKey = key.replaceAll("[^a-zA-Z0-9/_.-]", "_");
```

---

## 🎨 UI/UX Design Decisions

### 1. **Theme: FlatLaf IntelliJ**

**Why FlatLaf?**
- ✅ Modern, flat design
- ✅ Cross-platform consistency
- ✅ Dark/Light mode support (future)
- ✅ Native-like performance

**Color Palette:**
```java
SIDEBAR_COLOR = new Color(55, 58, 81);     // Dark blue-gray
BG_COLOR = new Color(245, 245, 247);       // Light gray
PRIMARY_COLOR = new Color(0, 122, 255);    // Blue
DELETE_COLOR = new Color(255, 59, 48);     // Red
```

### 2. **Icons & Visuals**

**Custom STOFOX Logo:**
- 🦊 Fox mascot (stofox-logo.png)
- Friendly and memorable
- Used in:
  - App icon (stofox.icns)
  - MainWindow title bar
  - PKG installer

**File Type Icons:**
```
📁 Folders
📄 Documents
🖼️ Images
📦 Archives
🎵 Audio
🎬 Video
```

### 3. **Accessibility**

**Keyboard Shortcuts:**
- Cmd+A: Select all
- Cmd+R: Refresh
- Delete: Delete selected
- Enter: Open/Download

**Screen Reader Support:**
```java
// Accessible labels
button.setAccessibleContext().setAccessibleName("Upload File");
table.setAccessibleContext().setAccessibleDescription("Object list");
```

---

## 🚢 Deployment: GitHub Actions CI/CD

### Automated Release Pipeline:

```yaml
name: Build and Deploy macOS PKG

on:
  workflow_dispatch:
    inputs:
      version:
        description: 'Version (e.g., 1.0.22)'
        required: true

jobs:
  build-and-deploy:
    runs-on: macos-latest
    steps:
      - Extract version
      - Update pom.xml & UpdateChecker.java
      - Build with Maven
      - Create macOS icon (iconutil)
      - jpackage → STOFOX-{version}.pkg
      - Upload to R2
      - Create GitHub Release
```

**Benefits:**
- ✅ One-click releases
- ✅ Version sync automation
- ✅ Consistent builds
- ✅ Automatic R2 deployment

---

## 📈 Roadmap: Gelecek Planları

### v1.1 (Q1 2025)
- [ ] Windows PKG support
- [ ] Linux AppImage
- [ ] Dark mode toggle
- [ ] Multi-bucket operations

### v1.2 (Q2 2025)
- [ ] Drag & drop upload
- [ ] File search & filtering
- [ ] Shareable public links
- [ ] Bandwidth throttling

### v1.3 (Q3 2025)
- [ ] Encryption at rest
- [ ] Custom metadata editor
- [ ] Lifecycle policies UI
- [ ] Team sharing features

### v2.0 (Q4 2025)
- [ ] Cloud sync (like Dropbox)
- [ ] Mobile companion app
- [ ] CLI tool
- [ ] API documentation

---

## 💡 Use Cases

### 1. **Static Website Hosting**
```
Upload HTML/CSS/JS → R2 bucket → Cloudflare CDN → Global users
```

### 2. **Media Library**
```
Upload photos/videos → Preview in STOFOX → Share via R2 URLs
```

### 3. **Backup Solution**
```
Daily backups → Folder upload → Version control → Disaster recovery
```

### 4. **Asset Delivery**
```
Game assets → R2 storage → Low latency downloads → Happy players
```

---

## 🤝 Katkıda Bulunun

STOFOX açık kaynak bir proje! Katkılarınızı bekliyoruz:

**GitHub:** [github.com/yourusername/stofox]

**Katkı Alanları:**
- 🐛 Bug reports
- 💡 Feature requests
- 🔧 Pull requests
- 📖 Documentation
- 🌍 Translations

---

## 🎓 Öğrendiklerimiz

STOFOX'u geliştirirken öğrendiğimiz dersler:

**1. Modern Java != Eski Java**
- Swing hala güçlü (FlatLaf ile)
- JRE embedding harika (jpackage)
- AWS SDK v2 çok temiz

**2. UX > Features**
- Kullanıcı dostu arayüz her şeyi yener
- Progress bars güven verir
- Error messages açık olmalı

**3. Automation Kazandırır**
- GitHub Actions zamandan tasarruf
- Auto-update kullanıcı memnuniyeti artırır
- Analytics gelişimi hızlandırır

**4. R2 is Underrated**
- S3'ten daha ucuz
- S3-compatible (kolay migrasyon)
- Cloudflare network = hızlı

---

## 🏁 Sonuç

**STOFOX**, Cloudflare R2 ekosisteminde eksik olan parçayı tamamlıyor: **kullanıcı dostu bir masaüstü istemcisi**.

Eğer:
- ✅ AWS S3 maliyetlerinden kaçıyorsanız
- ✅ Terminal komutlarından sıkıldıysanız
- ✅ Modern bir UI istiyorsanız
- ✅ Ücretsiz ve açık kaynak seviyorsanız

**STOFOX tam size göre!**

---

## 📥 Hemen Başlayın

**macOS Kullanıcıları:**
```bash
# Download latest PKG
curl -O https://r2client.brixyazilim.com/STOFOX-latest.pkg

# Install
open STOFOX-latest.pkg

# Launch
/Applications/STOFOX.app
```

**Developer'lar:**
```bash
# Clone repo
git clone https://github.com/yourusername/stofox
cd stofox

# Build
mvn clean package

# Run
java -jar target/STOFOX.jar
```

---

## 🙏 Teşekkürler

STOFOX'u mümkün kılan teknolojiler:

- ☁️ Cloudflare R2
- ☕ OpenJDK (Temurin)
- 🎨 FlatLaf
- 📊 Mixpanel
- 🔧 AWS SDK
- 🦊 Ve siz, topluluğumuz!

---

**Sorularınız mı var? Geri bildirim mi?**

📧 Email: hello@stofox.dev  
🐦 Twitter: @stofoxapp  
💬 Discord: [STOFOX Community]

**Happy Storing! 🦊**

---

*Not: STOFOX, Cloudflare'in resmi bir ürünü değildir. Cloudflare R2 ile uyumlu, bağımsız bir açık kaynak projedir.*
