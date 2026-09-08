# دليل النشر — A-HADDAD Dashboard 3.4.0 (مطابقة معايير ثيمات LuCI)

معالجة البنود T1–T13 من تقرير التدقيق. التاريخ: 2026-09-06

---

## الملفات المعدّلة (9 ملفات)

| # | الملف في المستودع | المسار على الراوتر |
|---|---|---|
| 1 | `files/usr/share/ucode/luci/template/themes/enan-dashboard/header.ut` | `/usr/share/ucode/luci/template/themes/enan-dashboard/header.ut` |
| 2 | `files/usr/share/ucode/luci/template/themes/enan-dashboard/footer.ut` | `/usr/share/ucode/luci/template/themes/enan-dashboard/footer.ut` |
| 3 | `files/usr/share/ucode/luci/template/themes/enan-dashboard/sysauth.ut` | `/usr/share/ucode/luci/template/themes/enan-dashboard/sysauth.ut` |
| 4 | `files/usr/share/ucode/luci/template/themes/enan-dashboard/version` | `/usr/share/ucode/luci/template/themes/enan-dashboard/version` |
| 5 | `files/www/luci-static/enan-dashboard/enan-dashboard.css` | `/www/luci-static/enan-dashboard/enan-dashboard.css` |
| 6 | `files/www/luci-static/enan-dashboard/enan-dashboard.js` | `/www/luci-static/enan-dashboard/enan-dashboard.js` |
| 7 | `files/www/luci-static/enan-dashboard/enan-navigation.js` | `/www/luci-static/enan-dashboard/enan-navigation.js` |
| 8 | `files/www/luci-static/enan-dashboard/dashboard-widgets.js` | `/www/luci-static/enan-dashboard/dashboard-widgets.js` |
| 9 | `files/etc/uci-defaults/30_luci-theme-enan-dashboard` | `/etc/uci-defaults/30_luci-theme-enan-dashboard` |

الصلاحيات: الملفات 1-8 بصيغة `644`، والملف 9 بصيغة `755`.

---

## خطوات التطبيق عبر WinSCP

### الخطوة 1 — الرفع
افتح جلسة WinSCP على الراوتر، وارفع كل ملف إلى مساره في الجدول أعلاه (استبدال الملف القديم).

### الخطوة 2 — ضبط الصلاحيات (من نافذة أوامر WinSCP: أوامر ← فتح الطرفية)
```sh
chmod 644 /usr/share/ucode/luci/template/themes/enan-dashboard/header.ut \
          /usr/share/ucode/luci/template/themes/enan-dashboard/footer.ut \
          /usr/share/ucode/luci/template/themes/enan-dashboard/sysauth.ut \
          /usr/share/ucode/luci/template/themes/enan-dashboard/version \
          /www/luci-static/enan-dashboard/enan-dashboard.css \
          /www/luci-static/enan-dashboard/enan-dashboard.js \
          /www/luci-static/enan-dashboard/enan-navigation.js \
          /www/luci-static/enan-dashboard/dashboard-widgets.js
chmod 755 /etc/uci-defaults/30_luci-theme-enan-dashboard
```

### الخطوة 3 — مسح الكاش وإعادة تشغيل الخدمات
```sh
rm -f /tmp/luci-indexcache /tmp/luci-modulecache
uci -q delete luci.main.index
uci -q commit luci
/etc/init.d/rpcd restart
/etc/init.d/uhttpd restart
```
> الأمر `uci delete luci.main.index` يزيل الخيار القديم الذي كان يضبطه السكربت السابق (اللوسي الحديث لا يقرأه أصلًا).

### الخطوة 4 — المتصفح
امسح كاش المتصفح أو افتح نافذة خاصة (Ctrl+Shift+N) ثم ادخل إلى الواجهة.

---

## قائمة الاختبار بعد التطبيق

| الاختبار | النتيجة المتوقعة |
|---|---|
| صفحة الدخول بدون إنترنت على جهازك | تظهر فورًا وبلا انتظار (أزيلت خطوط جوجل الخارجية) |
| تذييل صفحة الدخول | يظهر: Powered by LuCI (…) / OpenWrt (…) مع توقيع Haddad-inc كما هو |
| بعد الدخول — أعلى اليمين | اسم المستخدم الفعلي بدل "root" الثابتة، والحرف الأول منه في الصورة الرمزية |
| عنوان التبويب في المتصفح | `اسم-الجهاز - عنوان الصفحة \| A-HADDAD Dashboard` بدل القيم الثابتة |
| جهاز بدون كلمة مرور (حالتكم الحالية) | يظهر تحذير أصفر "لا توجد كلمة مرور!" مع رابط لإعدادها |
| النظام يعمل initramfs | يظهر تحذير وضع الاسترداد |
| تعطيل جافاسكربت في المتصفح | يظهر تحذير "جافاسكربت مطلوبة" |
| System → Language and Style → العربية | تُترجم أسماء القوائم وأزرار الحفظ وكل نصوص اللوسي (كانت تبقى إنجليزية) |
| الواجهة بالعربية (RTL) | القائمة الجانبية تنتقل لليمين، والاتجاهات تنعكس صحيحًا |
| صفحة Status | بطاقتا Memory وStorage تظهران حتى مع الواجهة العربية |
| لوحة المعلومات الرئيسية | كل الودجات تعمل كما كانت |
| قائمة الطوارئ (عند فشل تحميل القائمة) | لم يعد فيها رابط Firewall المعطّل |
| كسر الكاش بعد التحديث | يعمل تلقائيًا عبر `pkgs_update_time` (المعيار الرسمي) بدل `pkg_version` غير المعرّف |

---

## ما تم لكل بند

- **T1**: حُذفت روابط `fonts.googleapis.com` من `header.ut` و`sysauth.ut`؛ الثيم الآن يعتمد حزمة خطوط النظام القياسية (`--enan-font`) — نفس نهج ثيمات أوبن ورت الرسمية، يعمل أوفلاين بالكامل.
- **T2**: أُضيف سكريبت الترجمات القياسي `admin/translations` بالترتيب الرسمي (قبل `cbi.js`).
- **T3**: أُضيفت التحذيرات القياسية الثلاثة (كلمة المرور / initramfs / noscript) بنفس منطق الثيم الرسمي.
- **T4**: بقي توقيع Haddad-inc، وأُضيف بعده الإسناد الرسمي: `Powered by LuCI (…) / OpenWrt distname distversion (revision)` بروابطه القياسية — في صفحة الدخول والتذييل معًا.
- **T5**: اسم المستخدم الحقيقي من جلسة الدخول (`ctx.authuser`) بدل النص الثابت.
- **T6**: عُرّف `boardinfo` فعليًا داخل كل قالب (كان غير معرّف فكانت القوالب تعمل دائمًا على القيم الثابتة!)؛ القيم البديلة الآن عامة وغير مرتبطة بجهاز واحد، وأُسقطت ذاكرة 128MB الوهمية.
- **T7**: استُبدل `pkg_version` (غير الموجود في نطاق قوالب اللوسي) بـ `pkgs_update_time` — نفس معيار الثيم الرسمي لكسر الكاش.
- **T8**: أُصلح المنتقي `.alert` إلى `.alert-message` (فئة اللوسي الفعلية) مع تنسيق كامل لكل أنواع التنبيهات (warning/error/success/notice).
- **T9**: منطق بطاقات Memory/Storage الاحتياطية أصبح يستخدم دالة الترجمة `_()` ومفاتيح الكتالوج الرسمية، فيعمل بالعربية وكل اللغات.
- **T10**: حُذف رابط Firewall من قائمة الطوارئ (الفيرموير لا يحتويه).
- **T11**: دعم RTL كامل: القالب يضبط `dir="rtl"` تلقائيًا للغات ar/he/fa/ur، وكتلة CSS جديدة تعكس التخطيط (القائمة الجانبية، القوائم المنسدلة، النماذج، الجداول).
- **T12**: حُذف ضبط `luci.main.index` القديم من `30_luci-theme-enan-dashboard`؛ صفحة البداية تُحدَّد عبر ترتيب شجرة القائمة (لوحة المعلومات تبقى الأولى بترتيب 1).
- **T13**: أُضيفت ملاحظة صيانة رسمية في رأس ملف الـ CSS توثّق نقاط الاحتكاك مع أنماط اللوسي (`cbi-*` و`#modal_overlay`) لتسهيل المراجعة عند ترقية اللوسي. التفكيك الكامل لـ `!important` عملية كبيرة تُنفَّذ لاحقًا على جهاز فعلي.

## إصلاحات إضافية اكتُشفت أثناء التنفيذ

- `dashboard-widgets.js`: تصحيح تعبيرين نمطيين كانا لا يعملان (`/\\.([0-9]+)$/` → `/\.([0-9]+)$/`) — فلتر أسماء VLAN مثل `eth0.1`.
- `sysauth.ut`: ترميز HTML آمن لاسم المستخدم الافتراضي (`entityencode`).
- القوالب أصبحت تضبط `http.prepare_content('text/html; charset=UTF-8')` كما في الثيم الرسمي.
