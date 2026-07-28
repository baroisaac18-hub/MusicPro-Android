# MusicPro Player | مشغل MusicPro

A modern music player built with Kotlin + Jetpack Compose.
مشغل موسيقى عصري مبني بـ Kotlin + Jetpack Compose.

---

## Specifications | المواصفات

| Element | Detail |
|---|---|
| Package | `com.musicpro.player` |
| Language | Kotlin |
| UI | Jetpack Compose + Material3 |
| Compile SDK | 34 |
| Signing | V1 + V2 + V3 |
| Size | 48 MB |

---

## Features | المميزات

| Arabic | English |
|---|---|
| تشغيل الملفات الصوتية من الجهاز | Play audio files from device |
| خدمة تشغيل أمامية (Foreground Service) | Foreground service for background playback |
| إشعارات التحكم في التشغيل | Playback control notifications |
| التنقل بين الأغاني | Track navigation (next/previous) |
| تصفح المكتبة والبحث | Library browsing and search |
| حفظ جزء مفضل من الأغنية | Bookmark favorite part of a song |

---

## Tools & Technologies | الأدوات والتقنيات

| Technology | Usage |
|---|---|
| Kotlin | Primary programming language |
| Jetpack Compose | Modern declarative UI toolkit |
| Material3 | Adaptive and modern design system |
| MediaStore | Access audio files on device |
| Foreground Service | Background music playback |
| MediaMetadataRetriever | Extract audio file metadata |
| ContentResolver | Manage media content |
| Coroutines | Asynchronous operations |
| ViewModel | UI state management |

---

## Known Issue | مشكلة معروفة

Album art not showing for some externally downloaded tracks (e.g. from Facebook).
صور الألبوم لا تظهر لبعض الأغاني المحملة من مصادر خارجية (مثل فيسبوك).

### Root Cause | السبب

The app currently relies solely on MediaStore to load album art. Songs downloaded from external apps embed album art inside the file (ID3 tags) but are not registered in MediaStore.

### Solution | الحل

`MusicNotificationManager` already uses `MediaMetadataRetriever.getEmbeddedPicture()` successfully. Same approach can be applied to other screens.

```kotlin
fun loadAlbumArt(context: Context, song: Song): Bitmap? {
    val uri = song.albumArtUri
    if (uri != null && uri != Uri.EMPTY) {
        return try {
            MediaStore.Images.Media.getBitmap(context.contentResolver, uri)
        } catch (e: Exception) { null }
    }
    return try {
        val retriever = MediaMetadataRetriever()
        retriever.setDataSource(context, song.uri)
        val pic = retriever.embeddedPicture
        retriever.release()
        if (pic != null) BitmapFactory.decodeByteArray(pic, 0, pic.size) else null
    } catch (e: Exception) { null }
}
```

### Affected Files | الملفات المتأثرة

- `com/musicpro/player/data/MusicRepository.kt`
- `com/musicpro/player/ui/library/LibraryScreen.kt`
- `com/musicpro/player/ui/player/`
- All screens using `AsyncImage` or `Image` for album art

---

## Improvement Notes | ملاحظات تحسين

1. Missing `<uses-sdk>` — add explicit min/target SDK
2. `PreviewActivity` exported `true` — remove from release build
3. `ComponentActivity` exported `true` — unnecessary
4. Debug signing — use release signing for store distribution