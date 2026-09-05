# A-HADDAD Dashboard 3.3.0

## ملخص الإصدار

هذا الإصدار مبني على A-HADDAD Dashboard 3.2.0 دون حذف وظائفها السابقة. أضيفت طبقات إصلاح معزولة للنوافذ المنبثقة، وتخطيط حقول LuCI، وواجهات Logs، وبطاقات Memory وStorage.

## الإصلاحات

أضيفت مزامنة لحالة `#modal_overlay` مع `body.modal-overlay-active`، مع تعطيل `pointer-events` عندما تغلق LuCI النافذة. كما أضيفت مراقبة تغيّر class في body وجدولة فحص بعد النقر على أزرار الحفظ والإلغاء؛ وتبقى معالجات LuCI الأصلية هي المالكة لفتح وإغلاق النوافذ.

وُحّد عرض حقول الإدخال والقوائم المنسدلة إلى عرض أقصى قدره 620px وارتفاع 38px، مع ضبط `cbi-dropdown` و`ul.preview` والسهم الداخلي. بقيت `textarea` متعددة الأسطر مستثناة بارتفاعها الطبيعي. أضيفت كذلك قواعد pointer-events النهائية لمنع نافذة مغلقة من اعتراض النقرات.

أضيف `Kernel Log` إلى fallback navigation تحت مجموعة `Logs` مع `System Log`. أما المسارات الحقيقية في menu API فتظل مأخوذة من LuCI ولا تُستبدل إذا كانت متاحة.

تم تقوية fallback الخاص بـ Status بحيث لا يعتبر Memory أو Storage مكتملين بمجرد وجود العنوان أو جدول فارغ؛ لا يعتبر القسم مكتملًا إلا عند وجود `cbi-progressbar`. عند نقص البطاقة أو شريط التقدم تُضاف بيانات Total Available وUsed وCached وDisk space وTemp space وفق RPC النظام ونقاط التركيب.

## النسخة والملفات

رقم الثيم هو `3.3.0`. تمت مزامنة `building-files` مع الحزمة الرسمية تحت `enan-openwrt-package/package/luci-theme-enan-dashboard`. اسم الحزمة التقني بقي `luci-theme-enan-dashboard` للتوافق، بينما الاسم الظاهر هو `A-HADDAD Dashboard Theme`.

## الاختبارات المنفذة

نجحت اختبارات `node --check` لملفات JavaScript، والتحقق من JSON وShell، ومقارنة parity بين المصدر والحزمة. نجحت محاكاة hostname وLAN/WAN وswconfig وsysfs carrier، كما نجحت محاكاة modal المفتوح والمغلق: الحالة المغلقة تعطي `pointer-events: none` والحالة المفتوحة تعطي `pointer-events: auto`. تم أيضًا فحص عقود CSS الخاصة بالعرض والارتفاع والقوائم، والتحقق من وجود Kernel Log ورقم الإصدار.

لم يُنفّذ اختبار حي على راوتر فعلي من بيئة العمل؛ لذلك يجب بعد تثبيت الحزمة مسح cache المتصفح ثم تنفيذ `rm -f /tmp/luci-indexcache /tmp/luci-modulecache` وإعادة تشغيل `rpcd` و`uhttpd` إذا بقيت واجهة قديمة.

## البناء والتحديث

من جذر OpenWrt، يُنسخ مجلد الحزمة إلى `package/luci-theme-enan-dashboard`، ثم تُعاد metadata عبر `make prepare-tmpinfo V=s`، ويُفعل `CONFIG_PACKAGE_luci-theme-enan-dashboard=y` بواسطة `make menuconfig` أو `.config`. البناء المنفرد يتم عبر:

```sh
make package/luci-theme-enan-dashboard/clean
make package/luci-theme-enan-dashboard/compile V=s
find bin -type f -name 'luci-theme-enan-dashboard*' -ls
```

لإدخال الثيم في firmware الكامل:

```sh
make defconfig
make world -j$(nproc) V=s
```

لأن الثيم يستخدم CSS custom properties وJavaScript حديثًا، يحتوي Makefile على `LUCI_MINIFY_JS:=0` و`LUCI_MINIFY_CSS:=0` لتجنب إعادة تحليل الملفات بأدوات `jsmin` و`csstidy` القديمة.
