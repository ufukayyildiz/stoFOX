# STOFOX: The Missing Desktop Client for Cloudflare R2 🦊

## Why Every Developer Needs a Native R2 Client in Their Toolkit

In 2025, cloud storage isn't a luxury—it's a necessity. But finding the right tool? That's where the real challenge begins. If you're tired of AWS S3's high costs and want to migrate to Cloudflare R2's affordable storage, you'll hit a roadblock: **R2 doesn't have an official desktop client**.

Wrestling with terminal commands? Editing rclone config files? Trying to remember AWS CLI syntax every single time? It's exhausting.

That's why we built **STOFOX**.

---

## 📦 What is STOFOX?

**STOFOX** is a modern, user-friendly desktop application specifically designed for Cloudflare R2 object storage. Written in Java, distributed as a macOS PKG installer, and requiring zero installation thanks to its embedded JRE.

### Key Features:

✅ **Zero Dependencies**: One-click installation with embedded JRE  
✅ **Explorer-Style Interface**: Familiar experience like Windows Explorer or macOS Finder  
✅ **Fast File Operations**: Upload, download, delete, preview—all in a few clicks  
✅ **Folder Upload**: Bulk upload preserving folder hierarchies  
✅ **Auto-Update**: New versions automatically checked and installed  
✅ **Analytics**: Mixpanel integration for usage metrics and error tracking  
✅ **Modern UI**: Professional look with FlatLaf IntelliJ theme

---

## 🎯 The Problem: R2's Missing Link

Cloudflare R2 offers a fantastic alternative to AWS S3:
- **Zero egress fees** (no data transfer costs!)
- **S3-compatible API** (works with existing tools)
- **Global distribution** (Cloudflare's edge network)

But it has one problem: **No user-friendly desktop interface**.

### Current Solutions & Their Issues:

**1. Cloudflare Dashboard**
- ❌ Web-based (slow, limited)
- ❌ Bulk operations difficult
- ❌ No folder upload support

**2. AWS CLI / Rclone**
- ❌ Requires terminal knowledge
- ❌ Complex config files
- ❌ Difficult debugging

**3. Cyberduck / CloudBerry**
- ❌ Not optimized for R2
- ❌ Unnecessary S3 features
- ❌ Paid versions

### STOFOX's Solution:

✅ **Native Desktop App**: Optimized for macOS  
✅ **R2-First Design**: Only the features you need  
✅ **Open Source**: Open to community contributions  
✅ **Free**: Zero cost, zero ads

---

## 🏗️ Architecture: How It Works

STOFOX is built with modern software development principles:

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

**Why Java?**
- ✅ Cross-platform compatibility
- ✅ Mature S3 SDK ecosystem
- ✅ Memory-safe and secure
- ✅ JRE embedding support (jpackage)

**Why AWS SDK?**
- ✅ R2's S3-compatible API
- ✅ Battle-tested codebase
- ✅ Async/sync operation support
- ✅ Automatic retry mechanism

### 2. **UI/UX Philosophy**

STOFOX's interface is designed with the **"familiar yet modern"** principle:

**Hierarchical Navigation:**
```
My Bucket / photos / 2025 / january / 
[← Easy navigation with clickable breadcrumbs]
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
5. Table auto-refreshes
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

## 🚀 Features: Deep Dive

### 1. **Bucket Management**

**Automatic Bucket Discovery:**
```java
// Connect with R2 credentials
R2Config config = R2Config.loadFromFile();
S3Client client = S3Client.builder()
    .endpointOverride(config.getEndpoint())
    .region(Region.of("auto"))
    .credentialsProvider(...)
    .build();

// List all buckets
ListBucketsResponse response = client.listBuckets();
```

**Sidebar Navigation:**
- Bucket list auto-updates
- Bucket selection → load contents
- Quick bucket switching

### 2. **Object Operations**

**Upload:**
```java
// Single file
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
// Country: United States
// City: San Francisco
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

## 🔐 Security

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
button.getAccessibleContext().setAccessibleName("Upload File");
table.getAccessibleContext().setAccessibleDescription("Object list");
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

## 📈 Roadmap: Future Plans

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

## 🤝 Contributing

STOFOX is an open-source project! We welcome your contributions:

**GitHub:** [github.com/yourusername/stofox]

**Contribution Areas:**
- 🐛 Bug reports
- 💡 Feature requests
- 🔧 Pull requests
- 📖 Documentation
- 🌍 Translations

---

## 🎓 Lessons Learned

Lessons we learned while developing STOFOX:

**1. Modern Java != Old Java**
- Swing is still powerful (with FlatLaf)
- JRE embedding is great (jpackage)
- AWS SDK v2 is very clean

**2. UX > Features**
- User-friendly interface beats everything
- Progress bars build trust
- Error messages must be clear

**3. Automation Wins**
- GitHub Actions saves time
- Auto-update increases user satisfaction
- Analytics accelerates development

**4. R2 is Underrated**
- Cheaper than S3
- S3-compatible (easy migration)
- Cloudflare network = fast

---

## 🏁 Conclusion

**STOFOX** fills the missing piece in the Cloudflare R2 ecosystem: **a user-friendly desktop client**.

If you:
- ✅ Are escaping AWS S3 costs
- ✅ Are tired of terminal commands
- ✅ Want a modern UI
- ✅ Love free and open source

**STOFOX is for you!**

---

## 📥 Get Started Now

**macOS Users:**
```bash
# Download latest PKG
curl -O https://r2client.brixyazilim.com/STOFOX-latest.pkg

# Install
open STOFOX-latest.pkg

# Launch
/Applications/STOFOX.app
```

**Developers:**
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

## 🙏 Acknowledgments

Technologies that made STOFOX possible:

- ☁️ Cloudflare R2
- ☕ OpenJDK (Temurin)
- 🎨 FlatLaf
- 📊 Mixpanel
- 🔧 AWS SDK
- 🦊 And you, our community!

---

**Questions? Feedback?**

📧 Email: ufuk@devfox.net


**Happy Storing! 🦊**

---

*Note: STOFOX is not an official Cloudflare product. It's an independent open-source project compatible with Cloudflare R2.*



Client for Cloudflare R2

https://r2client.brixyazilim.com/STOFOX-latest.pkg
