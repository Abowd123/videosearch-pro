# VideoSearch Pro — نسخة بدون مفتاح Google API

هذا المشروع مُعدَّل ليعمل باستخدام **youtubei.js** (مكتبة YouTube.js) بدل
YouTube Data API الرسمي، فلا تحتاج مفتاح API ولا حصة استخدام يومية.

## البنية
```
outputs/
├── index.html                 ← الواجهة الأمامية (عدّلتها لتتصل بالسيرفر المحلي)
└── videosearch-backend/
    ├── server.js               ← سيرفر Node.js (البحث + التفاصيل + التنزيل)
    └── package.json
```

## خطوات التشغيل

### 1) تشغيل السيرفر الخلفي
يتطلب [Node.js](https://nodejs.org) نسخة 18 أو أحدث مثبّتة على جهازك.

```bash
cd videosearch-backend
npm install
npm start
```

سترى رسالة: `🚀 السيرفر يعمل على http://localhost:3000`

### 2) فتح الواجهة
افتح ملف `index.html` مباشرة في المتصفح (أو عبر أي سيرفر ملفات ثابتة
مثل `Live Server` في VS Code). طالما السيرفر الخلفي يعمل على المنفذ 3000،
سيتصل الموقع به تلقائيًا (`CONFIG.BACKEND_URL` في `index.html`).

## ما الذي تغيّر في index.html؟
- حُذفت الحاجة إلى `YOUTUBE_API_KEY` نهائيًا.
- البحث وتفاصيل الفيديو أصبحا يمرّان عبر `http://localhost:3000/api/search`
  و `http://localhost:3000/api/videos` بدل `googleapis.com`.
- التنزيل أصبح يفتح رابط بث مباشر من السيرفر المحلي
  (`/api/download?videoId=...&format=mp4|mp3`) بدل انتظار `download.php`.

## النشر على VPS خاص (Ubuntu/Debian) بدون دومين

```bash
# 1) تثبيت Node.js و PM2
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo bash -
sudo apt install -y nodejs nginx
sudo npm install -g pm2

# 2) رفع المشروع وتشغيله
git clone https://github.com/USERNAME/videosearch-backend.git
cd videosearch-backend && npm install
pm2 start server.js --name videosearch
pm2 save && pm2 startup   # نفّذ الأمر الذي يظهر لك لتفعيل التشغيل التلقائي

# 3) وضع index.html في مجلد Nginx
sudo mkdir -p /var/www/videosearch
sudo cp index.html /var/www/videosearch/
```

ثم أنشئ `/etc/nginx/sites-available/videosearch` بالمحتوى الموجود في
سجل المحادثة (reverse proxy لـ `/api/` مع `proxy_buffering off` ومهلة
قراءة أطول لتنزيلات الفيديو)، فعّله، وأعد تشغيل Nginx.

**بهذا الإعداد:** `CONFIG.BACKEND_URL` في `index.html` يبقى فارغًا `''`
لأن الواجهة والسيرفر على نفس النطاق (نفس الـ IP)، فلا مشاكل CORS ولا
Mixed Content.

## ⚠️ حدود مهمة يجب معرفتها
1. **صيغة mp3 الحالية هي في الواقع m4a** (أفضل مسار صوتي متاح من يوتيوب)
   وليست تحويلًا حقيقيًا لـ MP3. للتحويل الحقيقي، ثبّت `ffmpeg` وأضف
   `fluent-ffmpeg` كما هو موضّح في تعليق أسفل `server.js`.
2. **youtubei.js غير رسمية** (هندسة عكسية لواجهة يوتيوب الداخلية)، لذا قد
   تتوقف بعض الوظائف مؤقتًا إذا غيّرت يوتيوب طريقة عملها الداخلية.
3. `searchCache` في السيرفر (للتنقل بين صفحات النتائج) تُخزَّن في الذاكرة
   فقط وتُفرَّغ عند إعادة تشغيل السيرفر — مناسب لموقع شخصي/تجريبي فقط.
4. لنشر الموقع فعليًا على الإنترنت، يجب رفع `server.js` على سيرفر حقيقي
   (VPS، Render، Railway...) وتحديث `CONFIG.BACKEND_URL` في `index.html`
   ليشير لعنوانه بدل `localhost`.
