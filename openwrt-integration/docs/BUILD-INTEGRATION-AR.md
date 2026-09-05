# إدماج A-HADDAD Dashboard في بناء OpenWrt المتكامل

## 1. ما الذي تم إصلاحه؟

الثيم يحمل الآن الهوية المرئية الرسمية **A-HADDAD**. تم الإبقاء على الاسم التقني `luci-theme-enan-dashboard` ومسار الأصول `/luci-static/enan-dashboard` حفاظًا على التوافق مع ملفات ImageBuilder والإصدارات المثبتة سابقًا، لكن عنوان الحزمة في `menuconfig` ومفتاح التسجيل الجديد هما `A-HADDAD Dashboard Theme` و`AHADDADDashboard`.

يقرأ قسم **Network Interfaces** حالة الرابط من مصدر فيزيائي عندما يتوفر. ففي أجهزة swconfig تُستخدم نتيجة `luci.getSwconfigPortState` مع `network.getSwitchTopologies`، وفي الأجهزة التي لا تستخدم swconfig يُستخدم `/sys/class/net/<device>/carrier`. لذلك لا يُعتبر ارتفاع VLAN مثل `eth0.1` دليلًا على وجود كابل موصول. إذا لم يكشف النظام تسمية منفذ حقيقية، يعرض الثيم `Port N` بدل اختراع تسمية LAN أو WAN.

كما أضيف جلب hostname من `system.info` ثم `system.board` مع إعادة المحاولة بعد `DOMContentLoaded` وحدث `luci-loaded`. وأضيفت طبقة Cancel محايدة متوافقة مع LuCI وBootstrap للأزرار الأصلية والنوافذ التي تُنشأ ديناميكيًا، دون تغيير أزرار Save أو Apply أو الأزرار السلبية.

## 2. لماذا لا يكفي مجلد building-files؟

مجلد `building-files` يعمل كـ overlay عند استخدام ImageBuilder لأنه ينسخ الملفات مباشرة إلى rootfs، لكن `make menuconfig` لا يقرأ مستودع ملفات عاديًا ولا يحوله تلقائيًا إلى package. يجب نسخ مجلد الحزمة الموجود في هذا المشروع إلى شجرة OpenWrt، حيث يحتوي على `Makefile` يستخدم `feeds/luci/luci.mk` وعلى أشجار `htdocs` و`ucode` و`root`.

## 3. تجهيز شجرة OpenWrt

نفّذ الأوامر التالية من جذر OpenWrt، بعد وضع مجلد هذا المشروع في مسار معروف:

```sh
./scripts/feeds update -a
./scripts/feeds install -a

mkdir -p package/luci-theme-enan-dashboard
cp -a /path/to/enan-openwrt-package/package/luci-theme-enan-dashboard/. \
  package/luci-theme-enan-dashboard/
```

لا تضع الحزمة داخل `feeds/luci` إذا أردت الاحتفاظ بها أثناء تحديث feeds؛ وضعها مباشرة تحت `package/` مناسب للبناء المتكرر لعدة أجهزة.

## 4. تفعيل التبعيات في config

يحتوي `config/enan-dashboard.config` على خيار الثيم ومكونات التشغيل اللازمة. ادمجه بعد الاحتفاظ بإعدادات target/profile الموجودة لديك:

```sh
cp /path/to/enan-openwrt-package/config/enan-dashboard.config .config.enan
cat .config.enan >> .config
make defconfig
```

تتضمن الحزمة في `LUCI_DEPENDS` ما يلي: `luci-base` و`rpcd-mod-luci` ووحدات ucode (`fs` و`ubus` و`uci` و`rtnl`) ووحدات LuCI للإدارة والشبكة والحالة والنظام. ويتضمن fragment أيضًا `wpad` و`mesh11sd` وملفات ath79 المطلوبة للملف التعريفي المستخدم. إذا كانت شجرة OpenWrt تستخدم اختيارًا مختلفًا لبرنامج wireless، اترك resolver يقرر البديل المتوافق مع target بدل فرض حزمة متعارضة.

## 5. التحقق من ظهور الحزمة في menuconfig

افتح القائمة من جذر OpenWrt:

```sh
make menuconfig
```

ثم اذهب إلى:

```text
LuCI  --->  4. Themes  --->  <*> A-HADDAD Dashboard Theme
```

فعّل الحزمة بعلامة `*` وليس `M` إذا أردت إدماجها داخل firmware النهائي. في بعض فروع OpenWrt قد يظهر ترتيب التصنيفات باسم مختلف قليلًا، لذلك ابحث داخل `menuconfig` عن `A-HADDAD` أو عن معرف الحزمة `luci-theme-enan-dashboard`.

بعد الحفظ افحص أن الخيار موجود فعليًا:

```sh
grep -E '^CONFIG_PACKAGE_luci-theme-enan-dashboard=' .config
```

والنتيجة المطلوبة:

```text
CONFIG_PACKAGE_luci-theme-enan-dashboard=y
```

## 6. بناء الحزمة بصيغة IPK أو APK

لبناء الحزمة فقط، استخدم target الحزمة التقني كما هو معرف في مجلد `package/`:

```sh
make package/luci-theme-enan-dashboard/clean
make package/luci-theme-enan-dashboard/compile V=s
```

يختلف امتداد الناتج بحسب فرع OpenWrt ونظام إدارة الحزم المفعّل. اعثر على الناتج دون افتراض مجلد بعينه:

```sh
find bin/packages bin -type f \( \
  -name 'luci-theme-enan-dashboard_*.ipk' -o \
  -name 'luci-theme-enan-dashboard-*.apk' \
\) -print
```

قد يُكتب اسم APK بشرطة واسم IPK بشرطة سفلية بحسب أدوات الفرع؛ لذلك يمكن استخدام بحث أوسع:

```sh
find bin -type f -name 'luci-theme-enan-dashboard*' -print
```

بعد نجاح بناء الحزمة، ابنِ firmware الكامل:

```sh
make download -j$(nproc)
make world -j$(nproc) V=s
```

لا تحتاج إلى نسخ ملفات الثيم يدويًا إلى `files/` إذا فعّلت خيار الحزمة بعلامة `*`. ستدخل ملفات `htdocs` و`ucode` و`root` تلقائيًا في rootfs عبر `luci.mk`.

## 7. تثبيت IPK أو APK على جهاز سبق بناؤه

انسخ الحزمة إلى الجهاز عبر SCP أو WinSCP، ثم ثبّت الصيغة المناسبة فقط.

لصيغة IPK:

```sh
opkg install /tmp/luci-theme-enan-dashboard_*.ipk
```

لصيغة APK:

```sh
apk add --allow-untrusted /tmp/luci-theme-enan-dashboard-*.apk
```

بعد التثبيت، نظف cache وأعد تشغيل الخدمات:

```sh
/etc/init.d/rpcd restart
/etc/init.d/uhttpd restart
rm -f /tmp/luci-indexcache /tmp/luci-modulecache
```

ثم تحقق من التسجيل والمسارات:

```sh
uci show luci.themes
uci get luci.main.mediaurlbase
uci get luci.main.index
ls -l /www/luci-static/enan-dashboard/
ls -l /usr/share/ucode/luci/template/themes/enan-dashboard/
ls -l /usr/share/luci/menu.d/luci-theme-enan-dashboard.json
ls -l /usr/share/rpcd/acl.d/luci-theme-enan-dashboard.json
cat /usr/share/ucode/luci/template/themes/enan-dashboard/version
```

رقم النسخة الجديدة هو:

```text
3.3.0
```

إذا كان الجهاز قد أُقلع سابقًا ولم يُنفّذ سكربت التسجيل الأولي، نفّذه مرة واحدة فقط بعد مراجعة إعداداتك:

```sh
sh /etc/uci-defaults/30_luci-theme-enan-dashboard
/etc/init.d/rpcd restart
/etc/init.d/uhttpd restart
rm -f /tmp/luci-indexcache /tmp/luci-modulecache
```

## 8. اختبار الملفات يدويًا بصلاحيات WinSCP

إذا كنت تختبر تعديلًا منفردًا دون بناء حزمة، ضع الملف أولًا في `/tmp` ثم انسخه إلى مساره النهائي. يجب تنفيذ `chmod` على الملف بعد كل نسخ:

```sh
cp /tmp/enan-dashboard.css /www/luci-static/enan-dashboard/enan-dashboard.css
chmod 0644 /www/luci-static/enan-dashboard/enan-dashboard.css

cp /tmp/enan-dashboard.js /www/luci-static/enan-dashboard/enan-dashboard.js
chmod 0644 /www/luci-static/enan-dashboard/enan-dashboard.js

cp /tmp/dashboard-widgets.js /www/luci-static/enan-dashboard/dashboard-widgets.js
chmod 0644 /www/luci-static/enan-dashboard/dashboard-widgets.js

cp /tmp/header.ut /usr/share/ucode/luci/template/themes/enan-dashboard/header.ut
chmod 0644 /usr/share/ucode/luci/template/themes/enan-dashboard/header.ut

cp /tmp/sysauth.ut /usr/share/ucode/luci/template/themes/enan-dashboard/sysauth.ut
chmod 0644 /usr/share/ucode/luci/template/themes/enan-dashboard/sysauth.ut
```

ملفات JavaScript وCSS يجب أن تكون عادة `0644`، والمجلدات `0755`. لا تجعل ملفات الثيم `0777`. بعد النسخ نفّذ أوامر تنظيف cache وإعادة تشغيل `rpcd` و`uhttpd` السابقة، ثم اعمل hard refresh في المتصفح باستخدام `Ctrl+F5`.

## 9. تشخيص المنفذ الفيزيائي في CF-WA350

لا يمكن استنتاج تسمية LAN/WAN الحقيقية من لون بطاقة قديم أو من `network.interface dump` وحده. على الجهاز افحص topology وحالة switch:

```sh
ubus call luci getSwconfigFeatures '{"switch":"switch0"}'
ubus call luci getSwconfigPortState '{"switch":"switch0"}'
ubus call network.device status '{"name":"eth0"}'
```

وللأجهزة التي تعرض netdevs فعلية افحص carrier مباشرة:

```sh
for d in /sys/class/net/eth* /sys/class/net/lan* /sys/class/net/wan*; do
  [ -e "$d/carrier" ] || continue
  printf '%s carrier=' "${d##*/}"
  cat "$d/carrier"
done
```

النتيجة `carrier=1` تعني وجود رابط فيزيائي، و`carrier=0` تعني عدم وجوده. في swconfig يجب مطابقة `port` و`label` الناتجين من `network.getSwitchTopologies` مع نتيجة `getSwconfigPortState`. الثيم يعرض label الذي يقدمه النظام؛ إذا كان firmware لا يقدم `LAN 1` أو `WAN` فلن يتم تزوير هذه المسميات.

## 10. تشخيص خطأ getFeatures أو 404 بعد الدخول

تحقق من وجود object الخاص بـ LuCI:

```sh
ubus list | grep -E '(^|\.)luci($|[[:space:]])'
ubus call luci getFeatures
```

ويجب أن تكون `rpcd-mod-luci` مضمنة في firmware. أما خطأ `/admin/login` بعد تسجيل الدخول فيُعالج بتأكيد أن landing route هو:

```sh
uci get luci.main.index
```

والقيمة المطلوبة:

```text
/cgi-bin/luci/admin/enan-dashboard
```

إذا كانت القيمة صحيحة لكن route غير مسجل، تحقق من وجود الملف:

```sh
ls -l /usr/share/luci/menu.d/luci-theme-enan-dashboard.json
rm -f /tmp/luci-indexcache /tmp/luci-modulecache
/etc/init.d/rpcd restart
/etc/init.d/uhttpd restart
```

لا تُعد تشغيل `uci-defaults` على جهاز إنتاجي إذا كانت لديك قيمة مخصصة عمدًا لـ `luci.main.index` أو `mediaurlbase`.

## 11. قائمة الملفات التي تدخل الحزمة

تشمل الحزمة الرسمية ملفات JavaScript وCSS والأيقونة وواجهة dashboard تحت `htdocs/luci-static/enan-dashboard/`، وملفات العرض تحت `htdocs/luci-static/resources/view/enan-dashboard/`، وقوالب LuCI تحت `ucode/template/themes/enan-dashboard/`، بالإضافة إلى `menu.d` و`rpcd/acl.d` و`uci-defaults` تحت `root/`. تم تحديث نسخة الحزمة من `3.1.0` إلى `3.3.0` بعد إضافة إصلاحات hostname وphysical switch وCancel وتصغير الهيدر وتحسين modal وحقول CBI وLogs وMemory/Storage.

## 12. حدود الاختبار الحالي

تم التحقق محليًا من syntax لملفات JavaScript، وصحة JSON وShell، ومنطق hostname عبر DOM/RPC وهمي، ومنطق LAN/WAN عبر topology وهمية وحالة sysfs carrier وهمية. لم يُنفّذ اختبار حي على `192.168.100.22` في هذه الجولة لأن اتصال الجهاز غير متاح من بيئة العمل؛ لذلك يجب تنفيذ أوامر القسم التاسع على CF-WA350 للتأكد من أن firmware نفسه يصف المنافذ بالأسماء المتوقعة.

## المراجع

[1]: <https://github.com/openwrt/luci/blob/master/luci.mk> — ملف بناء حزم LuCI الرسمي.

[2]: <https://github.com/openwrt/luci/blob/master/modules/luci-base/htdocs/luci-static/resources/network.js> — تعريف `getSwitchTopologies` وبنية منافذ swconfig.

[3]: <https://github.com/openwrt/luci/blob/master/modules/luci-mod-network/htdocs/luci-static/resources/view/network/switch.js> — واجهة Switch المرجعية واستدعاء `getSwconfigPortState`.

[4]: <https://openwrt.org/docs/guide-developer/packages> — دليل OpenWrt العام للحزم والبناء.

## 13. ملاحظة حول رسائل JSMIN وCSSTidy

الثيم يستخدم JavaScript حديثًا وCSS custom properties مثل `--enan-bg` و`var(--enan-bg)`. لذلك تضبط الحزمة `LUCI_MINIFY_JS:=0` و`LUCI_MINIFY_CSS:=0`، فلا تحاول أدوات OpenWrt القديمة `jsmin` و`csstidy` إعادة تحليل ملفات الثيم. هذا لا يعطل تصغير بقية حزم LuCI؛ يؤثر فقط على حزمة A-HADDAD ويضمن بقاء ملفاتها كما اختُبرت. إذا ظهرت رسائل JSMIN أو `Invalid property in CSS3.0` بعد تحديث الحزمة، فهذا يعني أن Makefile القديم ما زال مستخدمًا ويجب إعادة نسخه كاملًا.
