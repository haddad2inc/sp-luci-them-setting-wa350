# تقرير إصدار A-HADDAD Dashboard 3.2.0

## الخلاصة

هذا الإصدار يكمل الجولة المطلوبة على ثيم LuCI المخصص لجهاز COMFAST CF-WA350. الهوية المرئية أصبحت **A-HADDAD**، مع الحفاظ عمدًا على الاسم التقني `luci-theme-enan-dashboard` ومسارات `enan-dashboard` حتى تبقى أوامر ImageBuilder والترقيات السابقة قابلة للاستخدام. هذا الفصل بين الاسم المرئي والمعرف التقني موثق داخل README ودليل الدمج.

## الإصلاحات المنفذة

أصبح قسم **Network Interfaces** يفضّل حالة swconfig الفيزيائية عبر `luci.getSwconfigPortState` و`network.getSwitchTopologies`، ويستبعد منفذ CPU وواجهات VLAN مثل `eth0.1` و`eth0.2` من قائمة المنافذ الفيزيائية. وعند عدم توفر swconfig، يقرأ `carrier` من `/sys/class/net/<device>/carrier` ويجعله أعلى أولوية من الحالة المنطقية. لذلك يمثل `carrier=0` منفذًا رماديًا حتى لو بقيت واجهة الشبكة المنطقية في حالة up.

يُحدّث hostname من `system.info` أولًا ثم `system.board`، مع إعادة المحاولة عندما لا تكون LuCI RPC قد اكتملت بعد، والاستماع إلى `luci-loaded`. وهكذا لا يبقى النص القديم في header عند الانتقال من لوحة Dashboard إلى صفحات Status أو Network أو System.

تم تصغير الهيدر إلى ارتفاع 52px، وتقليل الشعار إلى 28px، واسم العلامة إلى 15px، وhostname إلى 10px، وعناصر التنقل إلى 11px مع padding أصغر. أضيف أيضًا تصنيف ديناميكي لأزرار `Cancel` و`Close` التي تنشئها LuCI لاحقًا، مع طبقة CSS محايدة بنمط Bootstrap لا تغيّر أزرار Save وApply وAction وNegative.

تم تحديث شاشة الدخول إلى `A-HADDAD | Professional Networking Solutions`، وتحديث وصف ACL وتسجيل الثيم الأولي إلى مفتاح `AHADDADDashboard` مع إزالة المفتاح القديم `ENANDashboard` عند الإقلاع الأول. بقيت مسارات الأصول والقائمة التقنية كما هي للتوافق.

## الملفات المعدلة أو المضافة في المصدر

| المسار | الغرض |
|---|---|
| `www/luci-static/enan-dashboard/dashboard-widgets.js` | قراءة swconfig/sysfs وإظهار المنافذ الفيزيائية، وتحسين بيانات CPU والحرارة والذاكرة. |
| `www/luci-static/enan-dashboard/enan-dashboard.js` | hostname مع retry و`luci-loaded`، وتصنيف أزرار Cancel/Close ديناميكيًا. |
| `www/luci-static/enan-dashboard/enan-dashboard.css` | تصغير الهيدر وإضافة نمط Cancel المتأخر والمحايد. |
| `usr/share/ucode/luci/template/themes/enan-dashboard/header.ut` | الهوية A-HADDAD وعرض hostname. |
| `usr/share/ucode/luci/template/themes/enan-dashboard/sysauth.ut` | هوية شاشة الدخول الجديدة. |
| `usr/share/ucode/luci/template/themes/enan-dashboard/version` | رفع النسخة إلى `3.2.0`. |
| `usr/share/rpcd/acl.d/luci-theme-enan-dashboard.json` | وصف A-HADDAD مع صلاحيات RPC/sysfs اللازمة. |
| `etc/uci-defaults/30_luci-theme-enan-dashboard` | تسجيل `AHADDADDashboard` وحذف المفتاح القديم وتثبيت landing route. |
| `www/luci-static/enan-dashboard/enan-navigation.js` | ملف التنقل المخصص الموجود ضمن النسخة الكاملة. |
| `www/luci-static/resources/view/enan-dashboard/overview.js` | محمل صفحة Dashboard الموجود ضمن النسخة الكاملة. |

## ملفات تكامل البناء الرسمي

| المسار | الغرض |
|---|---|
| `package/luci-theme-enan-dashboard/Makefile` | تعريف حزمة LuCI رسمية تستخدم `feeds/luci/luci.mk` وتظهر في `menuconfig`. |
| `config/enan-dashboard.config` | fragment يفعّل الثيم وLuCI وucode وwireless وmesh للـ target المعني. |
| `README.md` | خطوات سريعة للبناء الكامل. |
| `docs/BUILD-INTEGRATION-AR.md` | الدليل العربي الكامل لأوامر feeds وmenuconfig وipk/apk والتثبيت وcp/chmod وتشخيص المنافذ. |

## نتائج الاختبار المحلي

نجحت فحوصات `node --check` لملفات `enan-dashboard.js` و`dashboard-widgets.js` و`enan-navigation.js`. كما نجحت فحوصات JSON لملفي ACL وmenu، وفحص `sh -n` لسكربت `uci-defaults`، وفحص `git diff --check`.

اختبار DOM/RPC الوهمي أكد أن `system.info.hostname` يتقدم على `system.board.hostname`، وأن fallback يعمل عند غياب القيمة الأولى. واختبار topology الوهمي أكد أن LAN غير موصول يظهر `carrier=false`، وأن WAN الموصول يظهر `carrier=true`، وأن منفذ CPU يُستبعد. واختبار sysfs الوهمي أكد أن `carrier=false` يتغلب على `logical up=true`. كما نجحت اختبارات تحويل الذاكرة والحرارة والتردد.

تمت مقارنة كل مجلدات الثيم المصدرية مع نظائرها داخل الحزمة الرسمية، بما فيها CSS وJS والقوالب وACL وmenu وuci-defaults، ونجحت المطابقة دون فروق.

لم يتم بناء ملف binary فعلي بامتداد `.ipk` أو `.apk` داخل بيئة العمل الحالية لعدم وجود OpenWrt buildroot كامل في sandbox. الحزمة الرسمية المصدرية جاهزة للنسخ إلى شجرة OpenWrt، وهناك تُنتج الصيغة الصحيحة تلقائيًا عبر `make package/luci-theme-enan-dashboard/compile V=s` بحسب فرع OpenWrt ونظام الحزم المستخدم. لا ينبغي اعتبار أي أرشيف مصدر ملف IPK/APK ثنائيًا جاهزًا للتثبيت.

## ملاحظة اختبار الجهاز

الاتصال الحي إلى `192.168.100.22` لم يكن متاحًا في هذه الجولة، لذلك لم يُدّعَ تحقق حي من mapping منافذ CF-WA350. استخدم أوامر `ubus call luci getSwconfigPortState` و`ubus call network.device status` الموجودة في الدليل للتأكد من labels التي يقدمها firmware. الثيم لا يختلق تسمية WAN أو LAN؛ يعرض label الصادر من topology، أو `Port N` عند غياب label حقيقي.

## المراجع

[1]: <https://github.com/openwrt/luci/blob/master/luci.mk> — ملف بناء حزم LuCI الرسمي.

[2]: <https://github.com/openwrt/luci/blob/master/modules/luci-base/htdocs/luci-static/resources/network.js> — بنية `getSwitchTopologies` الرسمية.

[3]: <https://github.com/openwrt/luci/blob/master/modules/luci-mod-network/htdocs/luci-static/resources/view/network/switch.js> — استخدام `getSwconfigPortState` في واجهة Switch.

## تحديث بناء الحزمة بعد سجل المستخدم

أظهر سجل البناء أن الحزمة نفسها بُنيت بنجاح وأُنتج الملف `luci-theme-enan-dashboard_26.239.09332~955ac1f_all.ipk`. الرسالة `JSMIN Error: Unterminated set in Regular Expression literal` ورسائل `CSSTidy Invalid property in CSS3.0` لم توقف البناء لأن أوامر الأدوات تعمل داخل شرط يسمح بالاستمرار، لكنها تعني أن أدوات التحليل القديمة لا تناسب ملفات الثيم الحديثة. لذلك تم تحديث `Makefile` بإضافة `LUCI_MINIFY_JS:=0` و`LUCI_MINIFY_CSS:=0` لهذه الحزمة فقط، حتى لا يُعاد تعديل JavaScript أو CSS أو إسقاط custom properties. تعطيل التصغير لا يؤثر على تصغير حزم LuCI الأخرى.
