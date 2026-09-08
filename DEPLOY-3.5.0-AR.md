# دليل النشر — A-HADDAD Dashboard 3.5.0 (تحسينات الداشبورد + موائمة الهاتف)

التاريخ: 2026-09-06 — يبني على 3.4.0 (يجب أن يكون 3.4.0 مثبتًا أولًا).

---

## الملفات المعدّلة في هذا الإصدار (8 ملفات)

| # | الملف في المستودع | المسار على الراوتر |
|---|---|---|
| 1 | `files/www/luci-static/enan-dashboard/dashboard-widgets.js` | `/www/luci-static/enan-dashboard/dashboard-widgets.js` |
| 2 | `files/www/luci-static/enan-dashboard/enan-dashboard.css` | `/www/luci-static/enan-dashboard/enan-dashboard.css` |
| 3 | `files/www/luci-static/enan-dashboard/enan-dashboard.js` | `/www/luci-static/enan-dashboard/enan-dashboard.js` |
| 4 | `files/www/luci-static/resources/view/enan-dashboard/overview.js` | `/www/luci-static/resources/view/enan-dashboard/overview.js` |
| 5 | `files/usr/share/ucode/luci/template/themes/enan-dashboard/header.ut` | `/usr/share/ucode/luci/template/themes/enan-dashboard/header.ut` |
| 6 | `files/usr/share/ucode/luci/template/themes/enan-dashboard/version` | `/usr/share/ucode/luci/template/themes/enan-dashboard/version` |
| 7 | `files/usr/share/luci/menu.d/luci-theme-enan-dashboard.json` | `/usr/share/luci/menu.d/luci-theme-enan-dashboard.json` |
| 8 | `files/usr/share/rpcd/acl.d/luci-theme-enan-dashboard.json` | `/usr/share/rpcd/acl.d/luci-theme-enan-dashboard.json` |

## خطوات التطبيق عبر WinSCP

```sh
# 1) ارفع الملفات أعلاه إلى مساراتها (استبدال).
# 2) الصلاحيات:
chmod 644 /www/luci-static/enan-dashboard/dashboard-widgets.js \
          /www/luci-static/enan-dashboard/enan-dashboard.css \
          /www/luci-static/enan-dashboard/enan-dashboard.js \
          /www/luci-static/resources/view/enan-dashboard/overview.js \
          /usr/share/ucode/luci/template/themes/enan-dashboard/header.ut \
          /usr/share/ucode/luci/template/themes/enan-dashboard/version \
          /usr/share/luci/menu.d/luci-theme-enan-dashboard.json \
          /usr/share/rpcd/acl.d/luci-theme-enan-dashboard.json
# 3) الكاش والخدمات (إعادة rpcd ضرورية لتطبيق صلاحيات ACL الجديدة):
rm -f /tmp/luci-indexcache /tmp/luci-modulecache
/etc/init.d/rpcd restart
/etc/init.d/uhttpd restart
# 4) امسح كاش المتصفح أو استخدم نافذة خاصة.
```

---

## ما تم مقابل طلباتك الثمانية

1. **الهاتف**: زر قائمة (☰) جديد في الهيدر يفتح القائمة الجانبية كدرج فوق المحتوى مع خلفية معتمة؛ على الهاتف (≤640px) تُخفى الخيارات العلوية بالكامل كما طلبت. يعمل مع العربية RTL (الدرج من اليمين).
2. **اهتزاز البطاقات**: البطاقات الأربع تُبنى مرة واحدة وتُحدَّث قيمها نصيًا فقط — لا إعادة رسم ولا حركة كل 5 ثوانٍ.
3. **سرعة المعالج N/A**: إذا لم يوفر الراوتر تردد المعالج، تتحول البطاقة تلقائيًا إلى **"Active Connections"** (عدد اتصالات conntrack النشطة/الأقصى) — مقياس احترافي لمشرفي الشبكات غير موجود في لوحات اللوسي الرسمية. (يتطلب صلاحيات ACL الجديدة — لذا إعادة rpcd).
4. **إجمالي التحميل/الرفع منذ الإقلاع**: يُحسب الآن من عدادات الأجهزة الفيزيائية الحقيقية (eth0 + wlan0 + wlan1 ...) فيظهر دائمًا، بدل الاعتماد على منافذ السويتش التي لا تملك عدادات.
5. **Wireless Load**: لكل واجهة ↓ سرعة تنزيل حية و↑ سرعة رفع حية (عينات عدادات البايت بين كل تحديثين) + سطر Σ بإجمالي المنقول تراكميًا، مع إبقاء معدل الرابط (link rate) كبيان إضافي.
6. **Network Interfaces**: المنافذ الفيزيائية الحقيقية بمسمياتها (من getBuiltinEthernetPorts على أجهزة DSA، ومن توپولوجيا السويتش على ath79: LAN/WAN)، المتصل يتلون بالأخضر مع سرعته، وأسفله إجمالي الرفع/التنزيل عندما يكون للمنفذ عدادات (أجهزة DSA مباشرة؛ وعلى السويتش عند تطابق VLAN واحد-إلى-واحد عبر eth0.<vid> — في وضع الجسر الحالي بالمنافذين داخل VLAN واحدة لا توجد عدادات منفصلة فيزيائيًا لكل منفذ، فتظهر "—" بصدق بدل أرقام مضللة).
7. **ميزات لمشرفي الشبكات**: لوحة جديدة **"Connected Devices"** تعرض الأجهزة المعروفة (الاسم، IP، MAC) مع شارة WiFi لمن يتصل لاسلكيًا — إضافة لبطاقة الاتصالات النشطة في البند 3.
8. **تبديل الثيم**:
   - عنصر Dashboard في القائمة أصبح مشروطًا بأن يكون ثيمنا هو النشط (`mediaurlbase`) — يختفي تلقائيًا مع الثيمات الأخرى بعد إعادة البناء/الإقلاع.
   - حارس في `overview.js`: إن فُتحت الصفحة وثيم آخر نشط (كاش قديم)، تُحوِّل تلقائيًا لصفحة الحالة الرسمية بدل الشاشة الفارغة.

## ملاحظات اختبار مقترحة
- الهاتف: افتح الواجهة وجرّب زر ☰ — الدرج + الإغلاق بالخلفية/Escape/اختيار رابط.
- وصّل كيبل في منفذ LAN وراقب اخضرار البطاقة وظهور السرعة.
- بدّل الثيم إلى Bootstrap من Language & Style ثم أعد الإقلاع → يختفي عنصر Dashboard؛ وافتح رابطها القديم مباشرة للتأكد من التحويل التلقائي.
