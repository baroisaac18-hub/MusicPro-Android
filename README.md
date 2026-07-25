# MusicPro Player

مشغل موسيقى مبني بـ Kotlin + Jetpack Compose.

## مواصفات التطبيق

| العنصر | التفاصيل |
|---|---|
| الحزمة | `com.musicpro.player` |
| اللغة | Kotlin |
| واجهة المستخدم | Jetpack Compose + Material3 |
| compileSdk | 34 |
| التوقيع | V1 + V2 + V3 |
| الحجم | 62 MB |

## المميزات

- تشغيل الملفات الصوتية من الجهاز
- خدمة تشغيل أمامية (Foreground Service)
- إشعارات التحكم في التشغيل
- التنقل بين الأغاني
- دعم تصفح المكتبة والبحث
- حفض جزء مفضل من اغنية 

---

## مشكلة معروفة: صور الألبوم لا تظهر لبعض الأغاني

### وصف المشكلة

الأغاني التي يتم تحميلها من مصادر خارجية (مثل فيسبوك) لا تظهر صورة ألبومها في شاشة المكتبة، رغم أن صورة الألبوم مضمنة داخل الملف الصوتي نفسه.

### السبب التقني

يعتمد التطبيق حاليا على MediaStore فقط لتحميل صور الألبوم:

```
// MusicRepository.kt - السطر الحالي
val albumArtUri = ContentUris.withAppendedId(
    Uri.parse("content://media/external/audio/albumart"),
    albumId
)
```

الأغاني المحملة من فيسبوك وتطبيقات خارجية تحفظ صورة الألبوم **مضمنة داخل الملف** (ID3 tags) لكنها **غير مسجلة** في جدول `album_art` الخاص بـ MediaStore. النتيجة: `albumArtUri` ترجع فارغة ولا تظهر أي صورة.

### ملاحظة إيجابية

كود `MusicNotificationManager` يستخدم بالفعل `MediaMetadataRetriever.getEmbeddedPicture()` ويستخرج الصورة المضمنة بنجاح للإشعارات. الآلية الصحيحة موجودة في التطبيق لكنها غير مطبقة في باقي الشاشات!

### الحل المقترح

```kotlin
// إضافة دالة مساعدة
fun loadAlbumArt(context: Context, song: Song): Bitmap? {
    // المحاولة الأولى: MediaStore
    val uri = song.albumArtUri
    if (uri != null && uri != Uri.EMPTY) {
        return try {
            MediaStore.Images.Media.getBitmap(context.contentResolver, uri)
        } catch (e: Exception) { null }
    }
    
    // المحاولة الثانية: استخراج الصورة المضمنة من الملف
    return try {
        val retriever = MediaMetadataRetriever()
        retriever.setDataSource(context, song.uri)
        val embeddedPicture = retriever.embeddedPicture
        retriever.release()
        if (embeddedPicture != null) {
            BitmapFactory.decodeByteArray(embeddedPicture, 0, embeddedPicture.size)
        } else null
    } catch (e: Exception) { null }
}
```

ثم استدعاء هذه الدالة في أماكن عرض صورة الألبوم (LibraryScreen, PlayerScreen, إلخ).

### الملفات المتأثرة

- `com/musicpro/player/data/MusicRepository.kt` - مصدر البيانات
- `com/musicpro/player/ui/library/LibraryScreen.kt` - شاشة المكتبة
- `com/musicpro/player/ui/player/` - شاشة المشغل
- جميع الأماكن التي تعرض `AsyncImage` أو `Image` لصورة الألبوم

## ملاحظات تحسين إضافية

1. **`<uses-sdk>` مفقود في AndroidManifest** - يفضل إضافة `minSdkVersion` و `targetSdkVersion` بشكل صريح
2. **`PreviewActivity` مصدرة `exported=true`** - أداة تطوير لا يجب أن تكون في إصدار نهائي
3. **`ComponentActivity` مصدرة `exported=true`** - غير ضروري
4. **توقيع Debug** - يجب استخدام توقيع release للنشر على المتاجر
