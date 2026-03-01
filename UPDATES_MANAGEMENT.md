# إدارة التحديثات - Updates Management

## نظام التحديثات الجديد

تم تحديث نظام التحديثات ليستخدم ملف `updates.json` أونلاين بدلاً من hardcoded links في الكود.

## ملف updates.json

يجب رفع هذا الملف على مستودع GitHub في المسار:
```
https://raw.githubusercontent.com/faithreborn/service-office-releases/main/updates.json
```

## كيفية تحديث الروابط

### 1. تعديل ملف updates.json محلياً

```json
{
  "service-office": {
    "version": "1.0.7",
    "download_url": "https://github.com/faithreborn/service-office-app/releases/download/v1.0.7/Service.Office.exe",
    "release_notes": "إصلاح مشاكل وتحسينات"
  }
}
```

### 2. رفع الملف على GitHub

```bash
# في مستودع service-office-releases
git add updates.json
git commit -m "Update: Service Office v1.0.7"
git push origin main
```

### 3. التحديث سيظهر فوراً

بمجرد رفع الملف، جميع المستخدمين سيرون التحديث الجديد عند فتح Launcher.

## المزايا

✅ **تحديث فوري**: لا حاجة لإعادة بناء Launcher
✅ **مركزي**: ملف واحد لجميع التطبيقات
✅ **سهل**: تعديل JSON بسيط
✅ **آمن**: لا rate limit من GitHub API

## الحلول المطبقة

### 1. قراءة رقم الإصدار الحقيقي ✅

تم إضافة دالة `get_app_version` التي تقرأ الإصدار من metadata الملف:

```rust
#[tauri::command]
fn get_app_version(exe_name: String) -> Result<String, String> {
    // يقرأ الإصدار من خصائص الملف في Windows
    // يستخدم PowerShell: (Get-Item 'path').VersionInfo.FileVersion
}
```

### 2. الروابط أونلاين ✅

تم تغيير `check_app_update` لقراءة من `updates.json`:

```rust
pub async fn check_app_update(app_id: &str, current_version: &str) -> Result<UpdateCheckResult, String> {
    let updates_url = "https://raw.githubusercontent.com/faithreborn/service-office-releases/main/updates.json";
    // يجلب الملف ويقرأ معلومات التطبيق
}
```

### 3. إغلاق التطبيق قبل التحديث ✅

تم إضافة دالة `kill_app_process`:

```rust
#[tauri::command]
pub fn kill_app_process(exe_name: String) -> Result<(), String> {
    // يستخدم taskkill /F /IM app.exe
}
```

وفي `App.tsx`:

```typescript
// إغلاق التطبيق قبل التحديث
await invoke('kill_app_process', { exeName: app?.exeName });
await new Promise(resolve => setTimeout(resolve, 1000)); // انتظار
// ثم التحميل والتثبيت
```

## الخطوات التالية

1. ✅ بناء Launcher الجديد
2. ✅ رفع `updates.json` على GitHub في مستودع `service-office-releases`
3. ✅ اختبار النظام
4. ✅ رفع إصدار جديد من Launcher

## ملاحظات مهمة

- يجب أن يكون ملف `updates.json` في برانش `main` في مستودع `service-office-releases`
- الرابط يجب أن يكون `raw.githubusercontent.com` وليس `github.com`
- عند رفع تطبيق جديد، حدّث `updates.json` فوراً
- الإصدار في `updates.json` يجب أن يطابق الإصدار في metadata الملف
