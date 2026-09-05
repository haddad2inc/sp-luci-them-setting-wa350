# A-HADDAD Dashboard — تكامل OpenWrt الكامل

هذه الحزمة تضيف ثيم **A-HADDAD Dashboard** إلى بناء OpenWrt المتكامل عبر `make menuconfig`. اسم الحزمة التقني محفوظ باسم `luci-theme-enan-dashboard` حتى لا تنكسر أوامر البناء والتثبيت القديمة، بينما الهوية المرئية ومفتاح الثيم الجديدان هما **A-HADDADDashboard**.

من جذر شجرة OpenWrt:

```sh
./scripts/feeds update -a
./scripts/feeds install -a

cp -a /path/to/enan-openwrt-package/package/luci-theme-enan-dashboard \
  ./package/luci-theme-enan-dashboard
cp /path/to/enan-openwrt-package/config/enan-dashboard.config ./.config.enan
cat .config.enan >> .config
make defconfig
make menuconfig
```

في القائمة اختر:

```text
LuCI  --->  4. Themes  --->  <*> A-HADDAD Dashboard Theme
```

يحتوي `LUCI_DEPENDS` في `Makefile` على مكونات LuCI وucode اللازمة، ولذلك يقوم OpenWrt بإظهار أو تفعيل التبعيات عند اختيار الحزمة. بعد حفظ `.config` يمكن بناء الحزمة منفردة أو إدماجها في الصورة الكاملة:

```sh
make package/luci-theme-enan-dashboard/compile V=s
make download -j$(nproc)
make world -j$(nproc) V=s
```

للعثور على ناتج البناء بصيغتي OpenWrt القديمة والحديثة:

```sh
find bin/packages bin -type f \( \
  -name 'luci-theme-enan-dashboard_*.ipk' -o \
  -name 'luci-theme-enan-dashboard-*.apk' \
\) -print
```

التعليمات العربية الكاملة، بما فيها الدمج، التثبيت على الجهاز، أوامر `cp` و`chmod`، وفحص الحالة الفيزيائية للمنافذ موجودة في:

```text
docs/BUILD-INTEGRATION-AR.md
```

لا تُحذف أي ملفات من `building-files` عند استخدام هذه الحزمة؛ بل تنسخ الحزمة إلى شجرة OpenWrt. أما اختبار ملفات الثيم مباشرة على جهاز سبق إقلاعه فيتم عبر أوامر النسخ والصلاحيات الموضحة في الدليل الكامل، ثم تنظيف cache وإعادة تشغيل `rpcd` و`uhttpd`.
