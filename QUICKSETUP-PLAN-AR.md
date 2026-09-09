# خطة التنفيذ والرؤية الكاملة — حزمة `luci-app-quicksetup`
**الإصدار:** 2.2 (قرارات Q1–Q4 محسومة + دمج findings سكربت المصنع `99-asu-defaults-general` الوافد في تحديث المستودع) · **التاريخ:** 2026-09-09
**المرجع:** مستودع `sp-luci-them-setting-wa350` (مراجعة كاملة) · الوايزرد الحالي (`wizard`) كنقطة انطلاق سلوكية · دراسات المنصات أدناه

---

## 0) الملخص التنفيذي

سنحوّل الوايزرد الحالي — الذي يعمل بكفاءة — من «ملفات overlay مبعثرة داخل `files/`» إلى **حزمة OpenWrt مستقلة وقابلة للاعتماد في base** باسم `luci-app-quicksetup`، تحقق أربعة أهداف متزامنة:

1. **رأس القائمة** (قرار Q1 المحسوم): عنصر `admin/quicksetup` بترتيب `order: 2` — بعد Dashboard (ترتيبه 1) مباشرة وقبل Status (10) وبقية أقسام LuCI — فتبقى لوحة المعلومات شاشة الدخول، وتكون «البرمجة السريعة» **أول عناصر الإعداد كلها**. ومع أي ثيم آخر غير A-HADDAD (حيث يختفي عنصر Dashboard المشروط بالثيم) تصبح Quick Setup تلقائيًا أول القائمة وشاشة الدخول — تدهور رشيد بلا كسر.
2. **نموذج موحّد قابل للتوسع**: «دور + إضافات» (Role + Add-ons) بدل 9 تركيبات مسطحة، يغطي كل ما يفعله الوايزرد الحالي + ما تفعله منصات MikroTik/airOS/SOHO (بوابة NAT، موسّع، ضيوف، منافذ ولوبات، كشف WAN تلقائي).
3. **توافق 100% مع قواعد OpenWrt**: نصوص مصدر إنجليزية + ترجمة `po/ar`، ACL أدنى صلاحية، لا أسرار على القرص (كلمة الروت عبر `luci setPassword` حيًّا)، خيارات netifd/hostapd قياسية فقط، حالة التطبيق في `quicksetup` وحده، تسجيل عبر `uci-defaults`، ومحرك procd بأوامر صريحة.
4. **سلامة تطبيق غير مسبوقة في الوايزرد الحالي**: لقطة احتياطية دائمة قبل كل تطبيق ← نافذة تأكيد 90 ثانية ← تراجع تلقائي (Revert) عند انقطاع الإدارة أو انتهاء العدّاد ← بطاقة «عنوانك الجديد» عند تغيّر IP ← استرجاع حتى بعد إعادة إقلاع في منتصف النافذة.

**النهج:** 4 مراحل (P0 مكافئ وظيفي كامل للوايزرد الحالي ببنية الحزمة الصحيحة ← P1 السلامة والخدمات ← P2 البوابة والمنافذ ← P3 احترافات الميدان). كل مرحلة تُسلَّم قابلة للبناء والاختبار على الجهاز الحقيقي (CF-WA350 أولاً).

---

## 1) مراجعة المستودع الكاملة

### 1.1 الجرد العام

| المكوّن | المسار | الحالة | الاستخدام في مشروعنا |
|---|---|---|---|
| وايزرد overlay | `files/etc/config/wizard` · `files/etc/init.d/wizard` · `files/www/luci-static/resources/view/wizard.js` · `menu.d/luci-app-wizard.json` · `acl.d/luci-app-wizard.json` | يعمل بكفاءة على WA350 (حسب إفادتكم) | **المرجع السلوكي** للحزمة الجديدة — نأخذ منطقه ونعيد بناءه بصيغة حزمة متوافقة |
| ثيم A-HADDAD | `files/www/luci-static/enan-dashboard/…` + قوالب ucode | إصدار 3.5.0، مُصلَّح وفق التدقيق (T1–T13) | **مرجع تغليف** (`openwrt-integration/package/luci-theme-enan-dashboard/Makefile` = قالب Makefile الجاهز الذي نحتذيه) |
| حزمة الثيم | `openwrt-integration/package/luci-theme-enan-dashboard/` | بنية صحيحة (root/htdocs/ucode + luci.mk) | **النمط الهيكلي** الذي نكرره لـ `luci-app-quicksetup` |
| بذرة الاعتماد | `openwrt-integration/config/enan-dashboard.config` | تعمل | نضيف نظيرًا لها `quicksetup.config` (بند 8) |
| سكربت المصنع | `files/etc/uci-defaults/99-asu-defaults-general` | **جديد (دفعة 2026-09-09)** — 230 سطرًا، سياسة المصنع الكاملة لأول إقلاع | **مصدر الحقيقة لبذرة الحزمة** — كل قيم `quicksetup` الافتراضية تُحاذى عليه (بند 1.4) |
| تقارير | `AUDIT-REPORT-AR.md` · `DEPLOY-3.4/3.5` · `RELEASE-3.3` | موثقة | بنود H3/H4/H5 وS4/S5 تُغلق **داخل الحزمة الجديدة** (بند 7) |
| ملفات المنصة | `files/etc/board.d/01_leds` · `02_network` · `hotplug.d/firmware/11-ath10k-caldata` | نسخ كاملة من ath79 + سطرا دعم cf-wa350 | خارج نطاق الحزمة (ملاحظة S1 في التدقيق تبقى قائمة كدين تقني منفصل) |
| آثار | `files/etc/wizerd/public.key` (فارغ، اسم خاطئ) · `98-llpd-setting` (خطأ إملائي) | — | تُنظَّف عند اعتماد الحزمة في base (لا تمس عمل الوايزرد) |

### 1.2 تقييم الوايزرد الحالي — ما نَحفظه كما هو (سلوك ميداني مُثبت)

منطق `etc/init.d/wizard` الحالي يُنتج هذه السلوكيات التي **يجب أن تبقى مطابقة** في المحرك الجديد (فحص المكافأة Parity في P0):

| # | السلوك الحالي | القرار في الحزمة الجديدة |
|---|---|---|
| B1 | `remove_old_config=1`: حذف كل واجهات/أجهزة `network` (عدا loopback) وكل `wifi-iface` ثم بناء من الصفر | يبقى — هو الوضع الافتراضي، مع خيار `merge` جديد (P1) لمن يريد تطبيقًا جزئيًا |
| B2 | بوابة افتراضية = `.1` من عنوان الإدارة، وDNS = البوابة، و`delegate=0` | يبقى (مع إتاحة override في تبويب الشبكة) |
| B3 | إعادة تسمية الراديوات `radio0/radio1` → `5G/2G` حسب `band` | يبقى **كخيار منتج** (قرار Q3) — المحرك الجديد يكتشف الراديوات بالـ`band` ديناميكيًا فلا يتأثر إن أُبقيت `radioN` |
| B4 | واجهتا بث: 2G باسم `SSID` و5G باسم `SSID 5G`، كلتاهما `wds=1` `isolate=1` `disassoc_low_ack=0` | يبقى — مع فصل `isolate` و`wds` كحقول في تبويب «البث والظهر» (رؤيتكم) بدل الفرض الأعمى |
| B5 | التشفير: `psk2` إن وُجد مفتاح وإلا `none` | **يتغير**: الافتراضي `psk2` (أو `sae-mixed` حسب wpad المتوفر)، و`none` تتطلب تأكيدًا صريحًا في المراجعة (إغلاق H5/G4) |
| B6 | واجهة mesh على 5G: `mesh_fwding=1` `mesh_rssi_threshold=0` وربطها بـ`lan` | يبقى + حقول mesh_id/enc/key تظهر شرطيًا مع إضافة `+mesh-backhaul` أو دور `mesh` |
| B7 | واجهة sta (كلينت) على 5G: `wds=1` `disassoc_low_ack=0` وربطها بـ`lan` | يبقى — أدوار `client-wds`/`repeater`/`ptp` + إضافة `+redundant-backhaul` لكلينت ثانٍ |
| B8 | VLAN: جهاز `8021q` فوق `br-lan` + جسر `br-vlan` + واجهة `vlan` ببروتوكول `none`، وربط البث بها عند أوضاع `*vlan*` | يبقى نصًا (نفس البنية المولّدة) مع إضافة `+vlan` |
| B9 | watchcat: `ping_reboot` `10m` `8.8.8.8` `forcedelay 30` + enable/restart، وإلا disable/stop | يبقى — حقوله تصبح قابلة للتعديل (تبويب الحساس) |
| B10 | زر الريسيت: كتابة `/etc/rc.button/reset` فارغة عند التعطيل، وحذف الملف عند الإلغاء | **يتحسن**: حفظ الأصل في `/etc/rc.button/reset.quicksetup-orig` واستعادته (لا حذف لملف النظام) + تأكيد مكتوب «أوافق» في الواجهة (إغلاق H3/G5) |
| B11 | كلمة الروت: تُخزن في UCI ثم `passwd` ثم حذفها | **يتغير جذريًا**: تطبيق حيّ عبر ubus `luci setPassword` من الواجهة مباشرة — لا تلمس القرص إطلاقًا (إغلاق H4/G3) |
| B12 | `_reverse_sync` عند الإقلاع: إن كان `wizard` بلا IP، يُعبأ من الإعداد الحي | يبقى ويُوسَّع كثيرًا (بند 4.6): استنتاج الدور والإضافات والقنوات والوضع من `network/wireless/dhcp` الحي حتى تعكس الواجهة الواقع دائمًا |
| B13 | التطبيق عبر procd trigger على تغيير `wizard` + `wifi reload` و`network reload` و`dnsmasq reload` مع `sleep` | **يتغير**: تطبيق صريح بأمر `apply` (لا مفاجآت عند أي commit)، وترتيب reloads مدروس بلا `sleep` عشوائي (إغلاق G12)، مع بقاء `service_triggers` للتوافق إن استُدعيت reload |
| B14 | SSID افتراضي `Haddad-WiFi` / mesh `Haddad-Mesh` / قنوات `auto` + `36` / `cell_density 0` | يبقى المبدأ (قيم بذرة في `etc/config/quicksetup` لا في الكود) مع **محاذاة القيم لسكربت المصنع** (F1/F4/F5/F6): `COMFAST-WA350` / `My-Mesh` / قنوات `1`+`157` / IP `192.168.100.22` — وقيم الوايزرد القديم تبقى مدعومة كخيار |

### 1.3 ما لا ننقله (مخالفات مسجلة في تدقيقكم نفسه)

- تخزين كلمة الروت في UCI (H4) · تشفير `none` افتراضيًا (H5) · حذف ملف زر الريسيت النظامي (H3) · نصوص عربية مصدرية داخل `_()` (S5/G11) · تعريف `rename_radio()` داخل كتلة شرطية وبمسافات (S4) · `uci commit dhcp` بلا تعديل dhcp (S4) · `sleep 3` بين reloads (S4) · عنوان القائمة hardcoded عربي (G15) · عنصر القائمة بترتيب 98 (الهدف: رأس قائمة الإعداد).

### 1.4 سكربت المصنع `99-asu-defaults-general` (وافد التحديث الأخير — سياسة المصنع التي نرثها)

أصبح في المستودع السكربت الذي ينتج **حالة الشحن الفعلية** للجهاز (أول إقلاع، OpenWrt 25/swconfig). قرأناه سطرًا سطرًا، وهذه سياساته ذات الصلة المباشرة بحزمتنا:

| # | سياسة المصنع (كما في السكربت) | أثرها على `luci-app-quicksetup` |
|---|---|---|
| F1 | LAN = `192.168.100.22/24` بوابة `192.168.100.1` · `delegate=0` · `dhcp.lan.ignore=1` | **بذرة الحزمة تُحاذى عليه** (كانت بذرة الوايزرد القديم `192.168.1.22` — تُصحَّح؛ ونموذج الرؤية لديكم يعرض `100.22` أصلًا) |
| F2 | السويتش: VLAN واحد `ports='0t 2 1'` ⇒ **المنفذان الفيزيائيان معًا في br-lan** (لا WAN) | discovery للمنافذ (P2) يقرأ عضوية `switch_vlan` الحقيقية؛ حالة المصنع = «كلا المنفذين جسر» — تبويب 6 يعرضها بصدق |
| F3 | إعادة تسمية الراديوات `radioN` → `5G/2G` حسب الـband | **تأكيد ميداني لقرار Q3** — نفس المنطق يعاد استخدامه في `lib-discovery` (بأسلوب نظيف) |
| F4 | راديوات: `country=US` · `htmode=HT20` · `beacon_int=200` · 5G: ch157/txpower28 · 2G: ch1/txpower27/noscan | المحرك **لا يمس أقسام الراديو إلا بما يختاره المستخدم** (C10) — قيم المصنع تبقى حيّة بعد أي تطبيق (rebuild يحذف الواجهات فقط لا الراديوات، كما في B1) |
| F5 | بث مفتوح: AP-5G `COMFAST-WA350 - 5G` + AP-2G `COMFAST-WA350` (encryption none) + **mesh `My-Mesh` مفتوح على 5G** · الكل `isolate=1` `wds=1` `disassoc_low_ack=0` | reverse-sync على جهاز مصنع ⇒ الدور المستنتج `ap` + إضافتا `wds` و`mesh-backhaul` (مصفوفة 4.6 تغطيه) · بذرة SSID/mesh تُحاذى لأسماء المصنع · الترقية إلى `psk2/sae` تبقى توصية التطبيق (B5) لا حالة المصنع |
| F6 | hostname `COMFAST-WA350` · timezone `<+03>-3` / `Asia/Aden` · **NTP محذوف** · ملاحظة `Haddad Inc - Developed by Eng. Salah` | بذرة النظام تُحاذى (تأكيد إضافي لـQ6) · حقل NTP في تبويب 2 افتراضيه «مفعّل» حسب نموذج رؤيتكم — تطبيقه يعيد ما حذفه المصنع (موثق كتحسين مقصود) |
| F7 | `passwd -d root` (بلا كلمة مرور) + dropbear `PasswordAuth=1` (بند H1 في تدقيقكم) | تبويب 2 (كلمة المدير الحية عبر `setPassword`) هو **أداة المعالجة الميدانية** لهذه الحالة؛ التحذير الأصفر في الثيم 3.4.0 يكمل الصورة. إصلاح المصنع نفسه قرار base منفصل عن الحزمة |
| F8 | الجدار الناري معطَّل وملفه مُزاح إلى `firewall.unused` (H2) + dnsmasq/odhcpd معطّلان | دور `router` (P2) يفحص وجود `/etc/config/firewall` فعليًا: غائب ⇒ يحذّر «لا يمكن إنشاء مناطق NAT بدون جدار ناري» ويتخطى Zones بدل الفشل الصامت؛ توصية الخطة بإعادة جدار أدنى في base تبقى قائمة |
| F9 | حذف `system.@button[0]` (تعطيل الريسيت من طبقة system) | تبويب 8 يدير الطبقتين معًا (قسم button في system + سكربت `/etc/rc.button/reset`) مع حفظ الأصل — أعمق من B10 |
| F10 | ترقيع `sed` لملفات حية (10_system.js/openwrt_release/os-release/bootstrap footer) (S2) | خارج نطاق الحزمة — الحزمة لا تكرر هذا النمط إطلاقًا (C6)؛ يبقى دينًا تقنيًا منفصلًا على base |

**خلاصة:** سكربت المصنع يمنحنا «الحالة صفر» الدقيقة التي يجب أن تعكسها شاشتنا عند أول فتح (عبر reverse-sync)، وتحاذي عليها بذرة `/etc/config/quicksetup` — فلا تضارب بين ما يشحن به الجهاز وما تقترحه البرمجة السريعة.

---

## 2) الدراسة العميقة: «الإعداد السريع» عبر كل المنصات

> الغرض: استخراج **المجموعة الفائقة (Superset)** التي تجعل شاشتنا الواحدة تغني المستخدم عن أي منصة أخرى، مع تبني أنماط UX/السلامة الأفضل من كل منها.

### 2.1 MikroTik RouterOS — QuickSet (المرجع الذهبي للمكانة والدور)

- **المكانة:** أول شاشة في WebFig فور الدخول — ليس «صفحة ضمن القائمة» بل **الشاشة الافتراضية**. نحاكي مكانتها بوضعها في رأس قائمة الإعداد (`order: 2` بعد Dashboard — قرار Q1؛ ومع الثيمات الأخرى تصبح هي الأولى فعليًا).
- **الأوضاع:** `Home AP` · `Home AP Dual` · `Basic AP` · `WISP AP` · `CPE` · `PTP Bridge AP` · `PTP Bridge CPE` · `CAP` (تُدار مركزيًا) — مع **مبدّل Router/Bridge** أعلى الصفحة يقلب السلوك كله (NAT+DHCP مقابل جسر).
- **الحقول (من توثيقهم الرسمي):**
  - *Internet:* Address Acquisition (Automatic/DHCP/PPPoE/Static) + MAC address + DNS.
  - *Wireless:* Network Name (SSID) · Frequency (auto/قائمة) · Band · Mode · Security Profile (WPA2/PSK) · WPS · Country.
  - *Local Network:* IP Address/Netmask · Gateway · DNS · **DHCP Server (مع NAT)** · Time Zone.
  - *VPN:* عميل L2TP/PPTP اختياري.
  - *System:* Identity (hostname) · Password (تُطبق فورًا).
- **أنماط نأخذها:** (1) اختيار الدور أولًا وكل شيء يتفرع عنه؛ (2) «Address Acquisition=Automatic» أي **كشف تلقائي لنوع WAN**؛ (3) Safe Mode الشهير لديهم (تراجع تلقائي عند فقدان الاتصال) — نحاكيه بنافذة التأكيد 90 ثانية.

### 2.2 Ubiquiti — airOS (الأقرب لمنتجاتكم الميدانية) + UniFi

- **airOS Setup Wizard (أول تشغيل):** Wireless Mode: `AP` · `AP-WDS` · `Station` · `Station-WDS` (+ وضع airMAX) ← **نفس أدواركم الحالية حرفيًا**؛ ثم SSID · Security (WPA2-AES) · **Country Code إجباري** · Channel Width (5/10/20/40MHz) · Frequency (قائمة مع إظهار قنوات DFS)؛ ثم Network Mode: `Bridge` / `Router` / `SOHO Router` (الأخير = NAT + DHCP عميل WAN)؛ Management IP (Static/DHCP) · Gateway · DNS؛ ثم System: Device Name · Admin User/Password · Timezone · **Ping Watchdog** (= watchcat لديكم) · airView (محلل طيف) · Discovery · SNMP.
- **أنماط نأخذها:** (1) Country + Channel Width + Frequency كحقول بث من الدرجة الأولى (G7)؛ (2) Ping Watchdog داخل المعالج نفسه؛ (3) محلل الطيف airView المرتبط باختيار القناة — نحققه في P3 عبر `iwinfo scan`؛ (4) أدوات Link Test للمدات (PTP).
- **UniFi Standalone:** معالج أول تشغيل مبسط: اسم WiFi + كلمة + country + إدارة سحابية اختيارية — درسهم: **البساطة القصوى في أول تشغيل**، التفاصيل لاحقًا.

### 2.3 SOHO: TP-Link · ASUS · Netgear · D-Link (نمط الخطوات المرشدة + الكشف التلقائي)

- **TP-Link Quick Setup:** خطوة الوقت/المنطقة ← **اختيار وضع التشغيل** (Router / AP / Range Extender / WISP / Client حسب الموديل) ← **كشف تلقائي لنوع الاتصال** (يختبر DHCP ثم PPPoE ثم يسأل Static) ← WiFi (SSID+كلمة، وSmart Connect لدمج النطاقين) ← شبكة ضيوف في موديلات ← **ملخص نهائي + إعادة تشغيل**.
- **ASUS QIS:** كشف تلقائي (DHCP/PPPoE/Static/PPTP/L2TP) ← **إنشاء حساب المدير وكلمة المرور إجباريًا في أول تشغيل** ← WiFi لكل نطاق (SSID/Passphrase/قناة/عرض) ← انتهاء. نمطهم الأهم: **لا يمكن تجاوز كلمة المدير**.
- **Netgear Genie / D-Link Wizard:** نفس الثلاثية (إنترنت ← واي فاي ← مدير) + خيار Guest Zone في D-Link + إعادة تشغيل في النهاية.
- **أنماط نأخذها:** (1) **كشف WAN تلقائيًا** قبل السؤال (S3)؛ (2) إجبارية كلمة مدير آمنة في أول تشغيل (نحققها كتوصية قوية + تحذير، لأن الإلزام المطلق قد يعطل الميدان — قرار Q5)؛ (3) ملخص نهائي قبل إعادة التشغيل؛ (4) خطوات مرقمة واضحة.

### 2.4 GL.iNet — First-run (نمط «أول دخول فقط»)

- معالج متسلسل يظهر **مرة واحدة عند أول دخول**: كلمة مدير إجبارية ← Timezone ← نوع الإنترنت (يُكتشف) ← WiFi ثنائي النطاق ← انتهاء، ثم لا يظهر مجددًا (يدخل مباشرة للوحة).
- **أنماط نأخذها:** (1) فكرة «first-run state»: نضيف `quicksetup.firstboot='1'` في البذرة؛ عند الدخول الأول تُبرز الواجهة شارة «إعداد أولي مقترح» مع قيم افتراضية معقولة، وبعد أول تطبيق ناجح تتحول الصفحة لوضع «تعديل سريع» العادي (لا إخفاء قسري — تبقى دائمًا أول القائمة كما طلبتم).

### 2.5 مجتمع OpenWrt — x-wrt/kiddin9 `luci-app-wizard` + iStoreOS Quickstart (مرجع التوافق القاعدي)

- **البنية القياسية** (فحصناها من المصدر مباشرة): `Makefile` (luci.mk + `conffiles /etc/config/wizard`) · `root/etc/init.d/wizard` (بأمر `reconfig` = reverse-sync يُستدعى من الواجهة عند `load()` عبر `fs.exec`) · `root/etc/uci-defaults/40-luci-wizard` · `menu.d` + `acl.d` · view بتبويبات (Net/Firmware/Wireless/Shortcuts) · `po/` لترجمات.
- **محركهم** يتعامل مع: `wan_proto` (dhcp/pppoe) · `lan_ipaddr/netmask/gateway/dns` · **`siderouter`** (بوابة جانبية: gateway لأعلى + `dhcp.lan.ignore=1` + `firewall.@zone[0].masq=1` + `network.wan.proto=none`) · **مبدّل IPv6 شامل** (ra/dhcpv6 hybrid ↔ معطل + delegate) · WiFi لكل راديو مع لاحقة `_5G/_2.4G` · فرض HTTPS (uhttpd/nginx/lighttpd) · `old_*` حقول لتغيير-فقط (delta apply).
- **أنماط نأخذها:** (1) **reverse-sync من الواجهة عند الفتح** (`fs.exec reconfig`) وليس فقط عند الإقلاع — أصدق تمثيلًا للواقع؛ (2) حقول `old_*` للمقارنة وتطبيق الفروقات فقط — نبني عليها **ملخص المراجعة** (قديم ← جديد) في S8؛ (3) نمط `siderouter` ≈ دور `ap` لدينا مع إدارة DHCP — نتبناه صريحًا في تبويب الخدمات؛ (4) مبدّل IPv6 المبسط (مطلوب مجتمعيًا لأي حزمة base — نضيفه P2)؛ (5) بنية الحزمة والملفات حرفيًا (هيكلنا في بند 4.1 مطابق لها + طبقاتنا الإضافية).
- **iStoreOS Quickstart:** درس مختلف — «الصفحة الأولى = لوحة حالة + إجراءات سريعة» (إعادة تشغيل، تغيير IP سريع). نأخذ منها **بطاقات الحالة الحية** أعلى شاشتنا (الدور الحالي، IP الحالي، الراديوات، وضع التطبيق الأخير) لتعمل الصفحة كـ«كوكتيل» حالة+إعداد.

### 2.6 Al-Malaki (مرجعكم العربي الميداني — من مواصفتكم/الفيديو 3.3)

أدوار AP / AP+VLAN / WDS مرسل / WDS مستقبل / Mesh · **أداة منافذ دخول/خروج لمكافحة اللوبات** · **محلل قنوات مدمج** · رفع/استعادة نسخة احتياطية · «وضع احترافي» · **ملخص مراجعة قبل التطبيق** ← كل هذه تدخل خطتنا صراحة: المنافذ (S6/P2) · محلل القنوات (P3) · النسخ الاحتياطي (P3 مع لقطة تلقائية من P1) · المراجعة (S8/P1) · الوضع الاحترافي = عرض diff كامل لكل تغيير (P3).

### 2.7 الخلاصة — المجموعة الفائقة الموحّدة

| الفئة | الاتحاد المستخرج من كل المنصات | مكانه عندنا |
|---|---|---|
| الأدوار | router(NAT) · ap(bridge) · ap-wds · client-wds/station · repeater(sta+ap) · mesh(802.11s) · ptp(وصلة موجهة) | تبويب 1 — 7 بطاقات |
| الإضافات | vlan · mesh-backhaul · guest · redundant-backhaul(+STP) · loop-roles(منافذ) · watchcat · ssh | تبويب 1 — شرائح multi-select |
| النظام | hostname · timezone · NTP · كلمة مدير (حيّة) · first-run | تبويب 2 |
| الشبكة | LAN proto(static/dhcp) · IP/mask/gw/dns · إدارة VLAN · STP · WAN(auto/dhcp/static/pppoe) · MTU · IPv6 | تبويب 3 |
| البث لكل نطاق | enable · SSID · أمان(psk2/sae-mixed/none+تأكيد) · مفتاح · قناة · عرض(htmode) · country · قدرة(txpower/cell_density) · isolate · wds · hidden | تبويب 4 (رسمي) + تبويب 5 (ميداني) |
| الظهر/الربط | mesh id/enc/key/fwding/rssi_threshold · sta ssid/enc/key/bssid·scan · كلينت احتياطي ثانٍ | تبويب 5 |
| المنافذ | uplink/link/isolated أدوار لكل منفذ مكتشف (DSA+swconfig) | تبويب 6 |
| الخدمات | DHCP server (حسب الدور) · DNS · LLDP · dnsmasq auto on/off · HTTPS إجباري اختياري | تبويب 7 |
| الحساس | تعطيل زر ريسيت (بتأكيد مكتوب + حفظ الأصل) · watchcat · SSH · remove_old_config · نسخ احتياطي | تبويب 8 |
| السلامة | ملخص diff · لقطة تلقائية · نافذة تأكيد 90s · تراجع تلقائي · بطاقة IP جديد · كشف WAN · محلل قنوات | تبويب 9 + المحرك |

---

## 3) الرؤية المنتجة (UX النهائي كما سيظهر للمستخدم)

صفحة واحدة `admin/quicksetup` بتسعة تبويبات (مطابقة لنموذج الرؤية HTML الذي أرفقتموه، بترتيبه وفلسفته):

```
[1 دور الجهاز] [2 النظام·رسمي] [3 الشبكة·رسمي] [4 الوايرلس·رسمي] [5 البث والظهر·مخصص]
[6 المنافذ واللوبات·مخصص] [7 الخدمات·رسمي] [8 الميداني والحساس] [9 المراجعة والتطبيق]
```

- **أعلى الصفحة شريط حالة حي** (من iStoreOS): الدور الحالي المستنتج · IP الإدارة · الراديوات المكتشفة · آخر تطبيق (تاريخ/نتيجة) · شارة «إعداد أولي» إن كان `firstboot=1`.
- **تبويب 1:** بطاقات الأدوار السبعة المصورة + شرائح الإضافات؛ الاختيار يفعّل/يُخفي الحقول شرطياً في بقية التبويبات (نفس ميكانيكا `depends` في form.js القياسي — بلا CSS مخصص ⇒ يعمل على bootstrap/argon/enan).
- **تبويب 4:** بطاقة لكل راديو **مكتشف فعليًا** (2.4/5/6 حسب الجهاز) — لا افتراض راديوين؛ الحقول الرسمية فقط.
- **تبويب 5:** هويتكم الميدانية: WDS · isolate · مستويات cell_density المبسطة · ظهر mesh · كلينت الظهر · الكلينت الاحتياطي الثاني (يفرض STP تلقائيًا — علاج سيناريو حلقة 16/17).
- **تبويب 8:** كل خيار حساس في **بطاقة تحذير حمراء**؛ تعطيل زر الريسيت لا يُفعَّل إلا بكتابة «أوافق».
- **تبويب 9:** جدول فروقات (قديم ← جديد) لكل قسم سيتغير **فعلًا** (نمط `old_*` من kiddin9) + ملاحظة «ستُلتقط لقطة احتياطية تلقائيًا» + زر [تطبيق] → **نافذة عدّاد 90 ثانية**:
  - IP لم يتغير ⇒ العدّاد في نفس الصفحة + استطلاع `status` كل 3 ثوانٍ + زر [تأكيد الإبقاء].
  - IP تغيّر ⇒ بطاقة كبيرة «عنوانك الجديد: http://x.x.x.x — سجّل الدخول هناك واضغط تأكيد خلال N ثانية وإلا سيتراجع الجهاز تلقائيًا».
  - انتهى العدّاد دون تأكيد ⇒ المحرك يستعيد اللقطة ويعيد reload (وريبوت إن فشلت الاستعادة الحية).
- **عربي/إنجليزي:** المصدر إنجليزي، والعربية عبر `po/ar` (تكتمل ترجمة التبويبات والتلميحات ورسائل المحرك المسجلة). RTL تتكفل به طبقة الثيم المُصلَحة (3.4.0).

---

## 4) البنية التقنية للحزمة

### 4.1 الشجرة الكاملة (بنمط حزمة الثيم الناجحة لديكم + نمط kiddin9)

```
openwrt-integration/package/luci-app-quicksetup/
├── Makefile                                    # luci.mk + conffiles + i18n (بند 4.2)
├── htdocs/luci-static/resources/
│   ├── view/quicksetup.js                      # الواجهة: 9 تبويبات (form.js قياسي)
│   └── quicksetup/                             # أصول مساعدة خفيفة (أيقونات SVG مضمنة data-URI — لا ملفات خارجية)
├── root/
│   ├── etc/config/quicksetup                   # بذرة الافتراضيات (بلا أسرار) + conffile
│   ├── etc/init.d/quicksetup                   # المحرك: extra_commands (status/apply/confirm/revert/snapshot/restore/migrate/detectwan)
│   ├── etc/uci-defaults/31-quicksetup          # تسجيل + هجرة wizard→quicksetup مرة واحدة
│   ├── usr/share/quicksetup/lib-discovery.sh   # اكتشاف الراديوات/المنافذ/التوبولوجيا (swconfig|DSA)
│   ├── usr/share/quicksetup/lib-build.sh       # مولّد UCI لكل دور/إضافة (network/wireless/dhcp/system/firewall)
│   ├── usr/share/quicksetup/lib-safety.sh      # snapshot/revert/postcheck/watchdog
│   ├── usr/share/luci/menu.d/luci-app-quicksetup.json     # order 2 (رأس قائمة الإعداد بعد Dashboard)
│   └── usr/share/rpcd/acl.d/luci-app-quicksetup.json      # أدنى صلاحية + file-exec
├── po/templates/quicksetup.pot
└── po/ar/quicksetup.po                         # luci.mk يبني luci-i18n-quicksetup-ar تلقائيًا
openwrt-integration/config/quicksetup.config    # بذرة .config للاعتماد في base
```

**لماذا 3 مكتبات sh منفصلة؟** محرككم الحالي ملف واحد 300 سطر يعمل، لكن مع 7 أدوار × 8 إضافات × منصات (swconfig/DSA) سيصبح 1500+ سطر. الفصل: `discovery` (يُقرأ فقط) · `build` (يُولد UCI فقط) · `safety` (لقطات/تراجع) — كل واحدة قابلة للاختبار بمعزل، و`init.d` يبقى منسّقًا نحيفًا.

### 4.2 Makefile (مسودة)

```make
include $(TOPDIR)/rules.mk

LUCI_NAME:=luci-app-quicksetup
PKG_NAME:=luci-app-quicksetup
PKG_VERSION:=1.0.0
PKG_RELEASE:=1

LUCI_TITLE:=Quick Setup wizard — role based one-page provisioning
LUCI_DEPENDS:=+luci-base +rpcd +rpcd-mod-luci +luci-mod-network +luci-mod-system +netifd +hostapd-utils
LUCI_PKGARCH:=all

PKG_LICENSE:=Apache-2.0
PKG_MAINTAINER:=A-HADDAD / Haddad-inc

define Package/luci-app-quicksetup/conffiles
/etc/config/quicksetup
endef

include $(TOPDIR)/feeds/luci/luci.mk
# call BuildPackage - OpenWrt buildroot signature
```

ملاحظات: `luci-i18n-quicksetup-ar` يُولَّد تلقائيًا من `po/` (نفس آلية كل تطبيقات LuCI الرسمية)؛ لا حاجة لـ`LUCI_MINIFY:=0` لأن الواجهة form.js قياسي (بخلاف الثيم)؛ التبعيات الدنيا فقط — `wpad` (بmesh أو بدونه حسب البناء) و`watchcat` و`lldpd` تُفحص وقت التشغيل ولا تُفرض (إن غابت يُخفى الحقل/يعطَّل الخيار مع تلميح).

### 4.3 القائمة وACL (مسودتان)

```json
// usr/share/luci/menu.d/luci-app-quicksetup.json
{
  "admin/quicksetup": {
    "title": "Quick Setup",
    "order": 2,
    "action": { "type": "view", "path": "quicksetup" },
    "depends": { "acl": [ "luci-app-quicksetup" ] }
  }
}
```
`"Quick Setup"` يترجم إلى «البرمجة السريعة» عبر `po/ar` (msgid في كتالوج التطبيق) — لا hardcoded عربي (إغلاق G15). الترتيب `2`: بعد Dashboard (`1`) وقبل Status (`10`) ⇒ **أول عناصر الإعداد، وتبقى لوحة المعلومات شاشة الدخول** (قرار Q1 المحسوم). ومع أي ثيم آخر يختفي عنصر Dashboard المشروط بالثيم فتصبح Quick Setup أول القائمة وشاشة الدخول تلقائيًا.

```json
// usr/share/rpcd/acl.d/luci-app-quicksetup.json
{
  "luci-app-quicksetup": {
    "description": "Grant access to Quick Setup",
    "read":  { "uci": [ "quicksetup", "wireless", "network", "dhcp", "system", "firewall", "watchcat" ],
               "ubus": { "network.wireless": ["status"], "iwinfo": ["scanlist","scaninfo","phyname"] } },
    "write": { "uci": [ "quicksetup", "wireless", "network", "dhcp", "system", "firewall", "watchcat" ],
               "ubus": { "luci": ["setPassword"] },
               "file": { "exec": { "/etc/init.d/quicksetup": ["status","apply","confirm","revert","snapshot","restore","migrate","detectwan"] } } }
  }
}
```
(صيغة `file.exec` تُطابق rpcd file plugin؛ تُثبَّت مقابل مصدر rpcd عند تنفيذ P0 — وإن رفضت نسخة rpcd القديمة القوائم، نستخدم `"*"` ثم نضيّقها. قراءة `firewall/watchcat` محاطة بـ`L.resolveDefault` في الواجهة فغياب الحزمة لا يكسر الصفحة.)

### 4.4 محرك التطبيق — آلة الحالة (قلب السلامة)

```
        [الواجهة تجمع النموذج] ──▶ uci set quicksetup.default.* (staged) ──▶ fs.exec apply
                                                                                    │
   ┌──────────────────────────────────────────────────────────────────────────┘
   ▼
(1) VALIDATE   تحقق كامل قبل لمس أي شيء: أدوار/حقول (ip4addr، wpakey، vid 1..4094،
               توافق mesh مع wpad المتوفر، منفذ uplink موجود…) — أي فشل = خروج بلا أثر
(2) SNAPSHOT   tar czf /etc/quicksetup/snapshots/<UTC-ts>.tar.gz  ← /etc/config كامل
               + حالة خدمات (enabled list) + نسخة /tmp سريعة؛ يُحتفظ بآخر 3 (دائم على الفلاش
               ⇒ ينجو من انقطاع كهرباء/ريبوت في منتصف النافذة)
(3) BUILD+COMMIT  lib-build.sh يولد التغييرات (delta أو rebuild حسب remove_old_config)
               عبر uci staging ثم commit واحد لكل ملف متغير (network/wireless/dhcp/system/firewall/watchcat)
(4) RELOAD     ترتيب مدروس بلا sleeps: wifi reload ⟵ reload_config (netifd) ⟵ dnsmasq
               (وحسب الدور enable/disable)؛ تغييرات المنافذ/VLAN في swconfig قد تحتاج
               network restart — يُكشف ويُعلَن للمستخدم مسبقًا في المراجعة
(5) POSTCHECK  خلال ≤20s: br-lan/واجهة الإدارة تحمل IP المطلوب؟ wifi up (ubus network.wireless
               status)؟ dnsmasq يخدم إن لزم؟ فشل ⇒ REVERT فوري
(6) CONFIRM WINDOW (90s)  marker: /tmp/run/quicksetup/pending + /etc/quicksetup/pending (دائم)
               الواجهة تستطلع status (عدّاد تنازلي)؛ [تأكيد] ⇒ fs.exec confirm ⇒ حذف markers
               + تسجيل «applied OK» في /tmp/quicksetup.log + (اختياري) حفظ snapshot كـ last-known-good
(7) TIMEOUT/FALSE ⇒ REVERT  استعادة اللقطة ⟵ reload ⟵ إن لم تعد الإدارة خلال 30s ⇒ reboot
(8) BOOT-GUARD   boot() يفحص /etc/quicksetup/pending: وُجد ⇒ REVERT تلقائي (طبّق ثم مات الجهاز
               قبل التأكيد) — وإلا reverse-sync (B12 الموسع) ولا تطبيق أبدًا عند الإقلاع
```

نقاط تصميم حاسمة:
- **لا procd auto-apply على كل commit** (بخلاف الحالي): التطبيق حدث صريح من الواجهة فقط — يمنع «إعادة تطبيق مفاجئة» عند أي تعديل يدوي لاحق، وهو شرط قبول مجتمعي. `service_triggers` تبقى مسجلة لاستجابة `reload` القياسية إن استدعاها المستخدم من صفحة Startup.
- **كلمة الروت لا تمر بالمحرك إطلاقًا**: الواجهة تستدعي `rpc.declare({object:'luci', method:'setPassword'})` مباشرة عند الحفظ (H4).
- **السجل:** كل خطوة تُسجل في syslog بوسم `quicksetup` + ملف `/tmp/quicksetup.log` (يظهر آخر 20 سطرًا في تبويب المراجعة بعد التطبيق — تشخيص ميداني فوري).
- **أوامر المحرك** (extra_commands في rc.common): `status` (JSON: pending؟ العداد؟ الدور المستنتج؟ آخر نتيجة؟ الراديوات؟ المنافذ؟) · `apply` · `confirm` · `revert` · `snapshot` · `restore <ts>` · `migrate` · `detectwan`.

### 4.5 طبقة التجريد العتادي (لماذا ستعمل الحزمة على أي راوتر)

- **الراديوات:** `lib-discovery.sh` يعدّ أقسام `wifi-device` ويقرأ `band`/`hwmode`/`channels` المتاح (من `iw phy` عند الحاجة) ⇒ تبويب 4 يُبنى ديناميكيًا (راديو واحد، اثنان، ثلاثة، 6GHz). إعادة التسمية `5G/2G` تطبق فقط عند `radio_naming='2G5G'` (قرار Q3) والمحرك لا يعتمد على الأسماء بل على الـband.
- **المنافذ/التوبولوجيا:** كشف نوع المنصة: وجود `switch` sections في network ⇒ **swconfig** (نمط WA350: `switch0` بمنفذين حسب `02_network` لديكم: `"0@eth0" "1:wan" "2:lan"`)؛ وإلا فحص `/sys/class/net/*/dsa` أو `bridge-vlan` ⇒ **DSA**. لكل نمط دوال: `ports_list` (اسم، حالة link، دور حالي) و`ports_apply_roles` (uplink/link/isolated). تبويب 6 يعرض القائمة الحقيقية المتصلة (أخضر=carrier) ولا يخترع أسماء — نفس فلسفة إصلاح الثيم 3.5.0 لديكم.
- **إشكالية WA350 المعروفة:** أدوار السويتش في board.d هي `wan/lan`، لكن **سكربت المصنع الجديد (F2) يلغيها عمليًا**: VLAN واحد `ports='0t 2 1'` يجمع المنفذين مع `eth0.1` داخل br-lan (وضع AP نقي بلا WAN). الطبقة تتعامل مع **أسماء الأجهزة الفعلية** وعضوية `switch_vlan`/`bridge-vlan` كما هي وقت التشغيل، وتترك التسميات اللطيفة للواجهة فقط.
- **wpad:** فحص `wpad-basic` مقابل كامل لميزات mesh/sae ⇒ تلميح للمستخدم قبل اختيار غير مدعوم (بدل فشل صامت).

### 4.6 reverse-sync الموسع + مصفوفة استنتاج الدور

تُنفَّذ `status`/`reconfig` قبل الرسم (نمط kiddin9 `fs.exec` عند `load()`):

| الدليل في الإعداد الحي | الاستنتاج |
|---|---|
| `network.wan.proto ∈ {dhcp,static,pppoe}` + zone wan/masq | role=`router` |
| واجهة `sta`+`ap` على نفس الراديو (وربط lan) | role=`repeater` |
| واجهة `sta` wds فقط بلا ap | role=`client-wds` |
| واجهة `mesh` + واجهة `ap` | role=`ap` + `mesh-backhaul` |
| واجهة `mesh` فقط | role=`mesh` |
| `ap` فقط + `network.vlan` موجودة | role=`ap` + `vlan` |
| `stp='1'` على br-lan + واجهتا sta | `+redundant-backhaul` |
| وجود `/etc/rc.button/reset.quicksetup-orig` | disable_reset=1 |
| `watchcat` enabled + قسم ping_reboot | `+watchcat` بقيمه |
| SSID/key/cell_density/channel لكل راديو، hostname، timezone، lan ip/mask/gw/dns | تعبئة الحقول المطابقة |
| لا شيء واضح (جهاز مصنع) | القيم من بذرة `/etc/config/quicksetup` + `firstboot=1` |

قاعدة ذهبية: **الواجهة تعرض الواقع دائمًا** — إن عدّل الفني شيئًا يدويًا من صفحات Network/Wireless، يظهر له في شاشتنا عند الفتح التالي (يمنع «العمى» الذي تسببه النماذج المنفصلة عن الواقع).

### 4.7 كشف WAN التلقائي (نمط SOHO/MikroTik «Automatic») — P2

`detectwan`: (1) `udhcpc -i <dev> -n -q -t 6 -T 3` اختبارًا ⇒ نجح: `dhcp` + يعرض ما حصل عليه؛ (2) فشل + وجود `pppoe-discovery`/رد PADI ⇒ يقترح `pppoe` (يسأل credentials)؛ (3) وإلا `static` مع حقول. المهلة ≤25s مع شريط تقدم في الواجهة، والنتيجة اقتراح قابل للتجاوز دائمًا (لا سحر صامت).

### 4.8 الترجمة (i18n)

- كل `_()` في `quicksetup.js` مصدرها إنجليزي؛ `po/templates/quicksetup.pot` يُولد بـ`make package/luci-app-quicksetup/{clean,compile}` أو أدوات luci `i18n-update`؛ `po/ar/quicksetup.po` نكتبه كاملًا (تبويبات، حقول، تلميحات، رسائل المحرك الظاهرة، عنوان القائمة).
- رسائل syslog تبقى إنجليزية (عرف مجتمعي) + مرآة عربية في `/tmp/quicksetup.log` اختيارية.
- بوابة P0: لا نص عربي واحد في الكود المصدري (`grep -P '[\x{0600}-\x{06FF}]'` على htdocs/root = صفر نتائج، عدا ملفات po).

### 4.9 مخطط `etc/config/quicksetup` (بذرة بلا أسرار)

```
config quicksetup 'default'
	option schema_version '1'
	option firstboot '1'
	option role 'ap'
	list addons 'wds'                    # wds|vlan|mesh-backhaul|guest|redundant|loop-roles|watchcat|ssh
	option radio_naming '2G5G'             # 2G5G|radioN  (Q3)
	option lan_ipaddr '192.168.100.22'     # محاذى لسكربت المصنع 99-asu (F1)
	option lan_netmask '255.255.255.0'
	option lan_gateway ''                  # فارغ ⇒ مشتق .1 = 192.168.100.1 (سلوك B2 = المصنع)
	option lan_dns ''
	option lan_proto 'static'              # static|dhcp
	option hostname ''                     # فارغ ⇒ يقرأ قيمة المصنع/النظام (COMFAST-WA350)
	option timezone '<+03>-3'              # محاذى للمصع (F6)
	option zonename 'Asia/Aden'
	option ntp '1'                         # رؤيتكم: مفعّل (المصنع يحذفه — تطبيقنا يعيده، موثق)
	# لكل نطاق (يملأ discovery الديناميكي الأقسام):
	option wifi_ssid 'COMFAST-WA350'       # هوية المصنع (F5) — لاحقة " 5G" تُشتق تلقائيًا (B4)
	option wifi_encryption 'psk2'          # psk2|sae-mixed|none(بتأكيد) — ترقية مقترحة فوق حالة المصنع المفتوحة
	option wifi_key ''
	option channel_2g '1'                  # محاذى للمصع (F4)
	option channel_5g '157'                # محاذى للمصع (F4)
	option htmode_2g ''                    # فارغ = لا يمس (قاعدي) — HT20 المصنعي يبقى حيًا
	option htmode_5g ''
	option country ''                      # لا تُفرض افتراضيًا (قاعدي) — US المصنعية تبقى حية حتى يختار المستخدم
	option cell_density '0'
	option isolate '1'
	option wds '1'
	option mesh_id 'My-Mesh'               # محاذى للمصع (F5)
	option mesh_encryption 'sae'           # توصية التطبيق (المصنع: none)
	option client_ssid ''
	option client_encryption 'psk2'
	option vlan_id '100'
	option uplink_port ''                  # يُعبأ من discovery
	option link_port ''
	option watchcat_mode 'ping_reboot'
	option watchcat_period '10m'
	option watchcat_host '8.8.8.8'
	option disable_reset_button '0'
	option remove_old_config '1'
	option confirm_timeout '90'
	# حالة (يكتبها المحرك فقط):
	option last_apply ''
	option last_result ''
```

---

## 5) كتالوج الحقول الكامل → أهداف UCI (الملحق التنفيذي)

> كل الحقول تكتب **خيارات netifd/hostapd/dnsmasq/fw4 القياسية فقط**؛ `quicksetup` يخزن النموذج+الحالة فقط (شرط التوافق #1).

**تبويب 2 — النظام:** hostname→`system.@system[0].hostname` · timezone→`system.@system[0].timezone`+`zonename` · NTP→`system.ntp.enabled`+`list server` (افتراضي خوادم OpenWrt) · كلمة مدير→ubus `luci setPassword` (لا قرص) · SSH→`dropbear.@dropbear[0].PasswordAuth/RootPasswordAuth`+enable (P2).

**تبويب 3 — الشبكة:** lan_proto→`network.lan.proto` (static|dhcp) · ipaddr/netmask→`network.lan.*` · gateway/dns→`network.lan.gateway/dns` (فارغ=مشتق/محذوف حسب الدور) · `delegate='0'` للجسر · STP→`network.br_lan.stp='1'` (يُفرض مع redundant/loop-roles ويُعرض مقفولًا) · VLAN إدارة→بنية B8 المولدة · MTU→`network.wan.mtu` (P2) · IPv6→نمط kiddin9 الشامل (P2) · WAN: proto→`network.wan.proto` + (dhcp: peerdns · static: ipaddr/netmask/gateway/dns · pppoe: username/password/ipv6=0) + device المكتشف.

**تبويب 4 — الوايرلس (لكل راديو R):** disabled→`wireless.R.disabled` · channel→`wireless.R.channel` · htmode→`wireless.R.htmode` (يُكتب فقط إن اختار المستخدم) · country→`wireless.R.country` (كذلك) · txpower→`wireless.R.txpower` (P3) · لكل واجهة بث N: mode='ap' · ssid · encryption (`psk2|sae-mixed|none`) · key · network (lan|vlan|guest) · hidden→`wireless.N.hidden` · isolate→`wireless.N.isolate` · wds→`wireless.N.wds` · disassoc_low_ack='0' (B4) · cell_density→`wireless.R.cell_density`.

**تبويب 5 — البث والظهر:** mesh: mode='mesh' + mesh_id + mesh_encryption(`sae|psk2|none`) + key + mesh_fwding='1' + mesh_rssi_threshold (B6) + الراديو المفضل · sta/backhaul: mode='sta' + ssid + encryption + key + wds='1' + bssid (اختياري P2) + زر [مسح الشبكات] (`iwinfo scan` — P3، وقبلها إدخال يدوي) · الكلينت الاحتياطي الثاني: قسم sta إضافي + فرض STP + تحذير اللوبات.

**تبويب 6 — المنافذ:** uplink_port/link_port → `quicksetup.default.*` (حالة) ⇒ المحرك يبني: DSA: عضوية `bridge-vlan` + pvid للعزل؛ swconfig: أقسام `switch_vlan` بعضوية المنافذ؛ الباقي معزول (خارج br-lan أو VLAN عزل).

**تبويب 7 — الخدمات:** DHCP server→`dhcp.lan.ignore` (0 لدور router / 1 لأدوار الجسر — سلوك منتجاتكم + نمط siderouter) · dnsmasq enable/disable حسب الدور · LLDP→`lldpd` enable+start (توافق MikroTik Neighbors — `98-llpd-setting` لديكم يصبح جزءًا من الحزمة) · syslog اختياري (P3).

**تبويب 8 — الحساس:** disable_reset_button→كتابة `/etc/rc.button/reset` stub + حفظ الأصل `.quicksetup-orig` (B10 المحسّن) · watchcat→أقسام `watchcat` (B9 مع حقول قابلة للتعديل) · remove_old_config (B1) · [نسخة احتياطية الآن]/[استعادة] → `sysupgrade -b` (P3) · SSH remote.

**تبويب 9 — المراجعة:** جدول diff يبنى من مقارنة `old_*` (تُلتقط عند load) مقابل القيم الجديدة + قائمة reloads المتوقعة + تحذير «سيتغير عنوان الإدارة» إن اختلف IP + العدّاد.

---

## 6) الهجرة من الوايزرد القديم (مرة واحدة، بلا فقدان)

`root/etc/uci-defaults/31-quicksetup` (يعمل أول إقلاع بعد الترقية/التثبيت):

1. إن وُجد `/etc/config/wizard` و`quicksetup.default.lan_ipaddr` فارغ ⇒ نسخ الحقول المشتركة مباشرة (lan_ipaddr/lan_netmask/wifi_ssid/wifi_key/mesh_*/client_*/channel_2g/channel_5g/cell_density/remove_old_config).
2. تحويل الوضع المسطح → دور+إضافات:

| الوضع القديم | الدور الجديد | الإضافات |
|---|---|---|
| `ap_wds` | ap | wds |
| `ap_wds_vlan` | ap | wds, vlan |
| `ap_wds_mesh` | ap | wds, mesh-backhaul |
| `ap_wds_vlan_mesh` | ap | wds, vlan, mesh-backhaul |
| `ap_wds_client` | repeater | wds |
| `ap_wds_vlan_client` | repeater | wds, vlan |
| `client_wds` | client-wds | wds |
| `mesh` | mesh | — |
| `ap_prodpand` | ap | wds (المحرك الحالي يعاملها كأبسط AP؛ تُوثَّق كملاحظة منتج) |

3. ضبط `firstboot='0'` (جهاز مهيأ مسبقًا) + `schema_version='1'` + تسجيل `migrated_from='wizard'` في الحالة.
4. **لا يحذف** `/etc/config/wizard` (يبقى للنسخ الاحتياطية/التراجع)، لكن الحزمة القديمة تُزال من base في الإصدار التالي لإصدار التعايش (قرار Q4 المحسوم). خلال إصدار التعايش يبقى القديم على order 98 كطوق نجاة؛ ولو بقي مثبتًا من overlay قديم: محركه يتفاعل فقط مع commit على `wizard` — ولا أحد يكتب عليه بعد الهجرة ⇒ تعايش آمن حتى أول sysupgrade.
5. **جهاز مصنع طازج (بلا `wizard` إطلاقًا):** لا هجرة ولا بذرة عمياء — `reverse-sync` يقرأ حالة المصنع الحية (F1–F10: `192.168.100.22`، بثا COMFAST-WA350 المفتوحان، mesh «My-Mesh»، المنفذان في VLAN1) ويعبئ النموذج منها، فتظهر الصفحة كانعكاس صادق للجهاز مع شارة `firstboot`.

---

## 7) قائمة التوافق مع قواعد OpenWrt (بوابة الاعتماد في base)

| # | القاعدة | التحقق |
|---|---|---|
| C1 | خيارات قياسية فقط في ملفات النظام؛ الحالة في `quicksetup` وحده | مراجعة lib-build.sh سطرًا سطرًا |
| C2 | لا أسرار على القرص: كلمة الروت عبر `setPassword`؛ مفاتيح WiFi في `wireless` (مكانها الطبيعي 600) فقط | grep على المحرك: لا `passwd` ولا تخزين |
| C3 | مصدر إنجليزي + po/ar + حزمة i18n مولدة | بوابة grep العربية (بند 4.8) |
| C4 | ACL أدنى صلاحية + file-exec مقيد بأوامر محددة | مراجعة acl.d مقابل ما تستدعيه الواجهة فعلًا |
| C5 | view محايد الثيم: form.js/قياسي، بلا CSS/خطوط/موارد خارجية | يعمل على bootstrap + argon + enan (اختبار 6) |
| C6 | تسجيل عبر uci-defaults فقط — لا sed على ملفات حية (S2) | الملف 31-quicksetup idempotent وقابل لإعادة التشغيل |
| C7 | procd init بأوامر صريحة + logging قياسي | لا sleeps، لا apply عند boot |
| C8 | لا reboot قسري افتراضيًا؛ reload ذري، وreboot معلن فقط عند تغييرات منافذ/VLAN أو فشل revert | رسائل المراجعة توضح مسبقًا |
| C9 | conffiles + PKG_LICENSE + MAINTAINER + LUCI_PKGARCH:=all + بنية root/htdocs/po الرسمية | مطابق لنمط حزمة الثيم لديكم |
| C10 | country/txpower/DFS لا تُفرض افتراضيًا (قاعدي تنظيمي) — اختيار المستخدم فقط | البذرة: `country ''` |
| C11 | الحزمة لا تكسر sysupgrade: conffile محفوظ + لا ملفات خارج المسارات القياسية + boot-guard يتعامل مع pending | اختبار sysupgrade من إصدار قديم |
| C12 | يغلق بنود التدقيق: H3 (زر ريسيت) · H4 (كلمة روت) · H5 (تشفير) · S4 (أسلوب المحرك) · S5/G11/G15 (i18n) | مصفوفة القبول |

---

## 8) خطة التنفيذ المرحلية (على مستوى الملفات)

### P0 — الهيكل + مكافئ وظيفي كامل للوايزرد الحالي (الأساس)
**النواتج:** شجرة 4.1 كاملة · Makefile يبني ipk نظيفًا · menu بترتيب 2 (رأس قائمة الإعداد، وDashboard تبقى شاشة الدخول) · ACL · بذرة config · `lib-discovery` (راديوات فقط) · `lib-build` (كل أدوار البث: ap/ap-wds/client-wds/repeater/mesh + إضافات vlan/mesh-backhaul — أي **كل الـ9 أوضاع القديمة**) · محرك apply/confirm مبسط (بلا نافذة تراجع بعد — تطبيق مباشر كالقديم) · reverse-sync الأساسي · الهجرة 31-quicksetup · `quicksetup.js` تبويبات 1/2(جزئي)/3/4/5/9(diff بسيط) · po/ar كامل للموجود.
**بوابة القبول:** على WA350: لكل وضع قديم من التسعة، تطبيق من الحزمة الجديدة ينتج **نفس مخرجات UCI** للمحرك القديم (سكربت مقارنة آلي `diff <(uci export network) <(...)` + wireless) + الدخول الفوري + العربية كاملة + موضعها في رأس قائمة الإعداد بعد Dashboard.
**معيار «الاعتماد في base» يتحقق هنا:** `openwrt-integration/config/quicksetup.config`:
```
CONFIG_PACKAGE_luci-app-quicksetup=y
CONFIG_PACKAGE_luci-i18n-quicksetup-ar=y
```
(يُدمج مع enan-dashboard.config أو seed موحد).

### P1 — السلامة والخدمات (قيمة المستخدم الأعلى)
**النواتج:** `lib-safety` كاملة (snapshot/revert/postcheck/watchdog/boot-guard) · نافذة تأكيد 90s + بطاقة IP الجديد + استطلاع status · تبويب 7 (DHCP/dnsmasq/LLDP حسب الدور) · تبويب 8 (زر ريسيت بتأكيد مكتوب + حفظ الأصل + watchcat قابل للتعديل + remove_old_config) · hostname/timezone/NTP (تبويب 2 كامل) · تشفير psk2/sae-mixed افتراضي + none بتأكيد صريح · firstboot state.
**البوابات:** اختبارات القبول 2/3/5/7 (انقطاع أثناء العداد ⇒ تراجع · IP يتغير ⇒ البطاقة · عربية بلا نصوص شاردة · هجرة من wizard).

### P2 — بوابة NAT + المنافذ واللوبات
**النواتج:** دور `router` كامل (WAN dhcp/static/pppoe + `detectwan` التلقائي + firewall zones/masq إن وُجد fw4 + DHCP server + IPv6 toggle بنمط kiddin9) · `lib-discovery` منافذ (swconfig + DSA) · تبويب 6 (uplink/link/isolate) · `+redundant-backhaul` (sta ثانٍ + STP قسري) · شبكة ضيوف `+guest` (واجهة + network + dhcp range + zone) · SSH flag.
**البوابات:** اختبار القبول 1 على منصتين (WA350 + جهاز DSA من أسطولكم أو QEMU x86/armsr) + اختبار 4 (حلقة 16/17 ⇒ STP يحجب).

### P3 — احترافات الميدان
**النواتج:** محلل قنوات/مسح شبكات (`iwinfo scan` لزر [مسح] في sta ولتوصية قناة البث) · نسخ احتياطي/استعادة UI (`sysupgrade -b` تنزيل/رفع) · htmode/country/txpower/DFS متقدم · «وضع احترافي» (diff كامل لكل سطر UCI) · 6GHz جاهزية · توثيق مستخدم عربي نهائي (دليل ميداني) + دليل صيانة.
**البوابة:** مصفوفة القبول كاملة (بند 10).

> التسلسل يضمن أن **كل مرحلة تُشحن صالحة للإنتاج**: P0 = ما لديكم اليوم لكن كحزمة صحيحة؛ P1 = يقفز بالسلامة فوق أي منصة درسناها؛ P2/P3 = التوسع.

---

## 9) سجل المخاطر

| الخطر | الاحتمال/الأثر | العلاج المصمم مسبقًا |
|---|---|---|
| انقطاع جلسة المستخدم عند تغير IP (سيناريو 16/17) | عالٍ/عالٍ | بطاقة العنوان الجديد + نافذة تأكيد على IP الجديد + revert تلقائي عند الفشل + boot-guard |
| ريبوت/كهرباء أثناء نافذة التأكيد | متوسط/عالٍ | marker دائم على الفلاش + revert عند الإقلاع التالي |
| اختلاف swconfig (WA350) عن DSA في أدوار المنافذ | عالٍ/متوسط | طبقة discovery + إخفاء تبويب 6 عند توبولوجيا غير معروفة + اختبار على منصتين في P2 |
| فيرمويركم الحالي يعطل الجدار الناري ويخفي ملفه (H2) | مؤكد/متوسط | دور router يكتب zones **إن وُجد** fw4 وإلا يحذر؛ ويوصى بإصلاح H2 في base بشكل منفصل (شرط ضمني لاعتماد P2) |
| تعارض المحرك القديم (overlay) مع الجديد | منخفض/عالٍ | الهجرة مرة واحدة + المحرك القديم يتفاعل فقط مع commit على `wizard` ولا أحد يكتبه + تعايش إصدارًا واحدًا ثم إزالة (قرار Q4 المحسوم) |
| تباعد بذرة الحزمة عن حالة المصنع (`99-asu-defaults-general`) | متوسط/متوسط | البذرة محاذية للمصع حرفيًا (جدول F1–F10) + reverse-sync يعرض الواقع دائمًا + بوابة اختبار: «جهاز مصنع طازج ⇒ فتح الصفحة يعرض قيَم المصنع كاملة بلا مفاجآت» |
| wpad-basic بلا mesh/sae على بعض البناءات | متوسط/متوسط | كشف وقت التشغيل + تعطيل الخيار مع تلميح (بدل فشل صامت) |
| مساحات فلاش صغيرة للقطات | منخفض/منخفض | اللقطة = /etc/config مضغوط (~15-40KB) × 3 نسخ — مهمل الحجم |
| كسر ترجمات/RTL مع كل إصدار LuCI | منخفض/متوسط | form.js قياسي فقط + بوابة grep + اختبار الثيمات الثلاثة |
| «تطبيق مباشر» القديم اعتاده الفنيون (زر واحد بلا عدّاد) | متوسط/منخفض | `confirm_timeout` قابل للضبط + وضع «تطبيق سريع» موثق (يكتب confirm مسبقًا) لمن يريد السلوك القديم |

---

## 10) مصفوفة اختبار القبول (نهائية)

1. **مكافأة P0:** الأوضاع التسعة القديمة ⇒ نفس UCI المُنتَج (مقارنة آلية) على WA350.
2. **كل دور × منصة:** ap/repeater/client-wds/mesh على WA350 (swconfig) + router/repeater على جهاز DSA (أو QEMU armsr/x86-64) ⇒ تطبيق + دخول فوري.
3. **العدّاد:** قطع الاتصال/إغلاق المتصفح أثناء 90s ⇒ revert كامل وعودة IP القديم دون تدخل.
4. **تغيّر IP:** البطاقة تعمل، تأكيد من العنوان الجديد يثبّت الإعداد.
5. **boot-guard:** تطبيق ثم `reboot` فورًا قبل التأكيد ⇒ الإقلاع التالي يستعيد اللقطة.
6. **اللوبات:** +redundant على سيناريو حلقة ⇒ STP يحجب المنفذ ولا ينهار الجسر.
7. **الهجرة:** جهاز عليه `wizard` قديم مهيأ ⇒ القيم تظهر صحيحة مرة واحدة، ولا تطبيق تلقائي بعد الترقية.
8. **i18n:** عربية كاملة (بلا msgid ظاهر) + إنجليزية كاملة + RTL سليم.
9. **الثيمات:** bootstrap / argon / enan-dashboard ⇒ لا كسر تنسيق.
10. **sysupgrade:** من إصدار فيه الحزمة ⇒ `/etc/config/quicksetup` محفوظ (conffile) والصفحة تعمل فورًا.
11. **الحيادية:** تعديل يدوي من Network/Wireless ثم فتح quicksetup ⇒ الواجهة تعكس الواقع (reverse-sync).
12. **الأمان:** لا أثر لكلمة الروت في أي ملف؛ `wireless` صلاحياته 600؛ none يتطلب تأكيدًا؛ زر الريسيت يُستعاد أصله عند الإلغاء.

---

## 11) نقاط القرار — **محسومة بتاريخ 2026-09-09**

| # | السؤال | القرار المعتمد ✅ |
|---|---|---|
| **Q1** | الاسم والترتيب | **محسوم:** `luci-app-quicksetup` بعنوان «Quick Setup / البرمجة السريعة» بترتيب `order: 2` — **Dashboard تبقى شاشة الدخول**، والبرمجة السريعة أول عناصر الإعداد بعدها مباشرة (وتصبح الأولى فعليًا مع الثيمات الأخرى التي تخفي عنصر Dashboard المشروط) |
| **Q2** | دور router/NAT | **محسوم:** P2 كما في الخطة — P0/P1 يحققان المكافأة الكاملة للوايزرد الحالي + السلامة أولًا |
| **Q3** | تسمية الراديوات | **محسوم:** إبقاء إعادة التسمية `5G/2G` افتراضيًا (`radio_naming='2G5G'` في البذرة) — والمحرك محايد يعمل بـ`radioN` أيضًا عبر خيار البذرة |
| **Q4** | الوايزرد القديم | **محسوم:** تعايش إصدارًا واحدًا (الجديد في رأس القائمة + القديم يبقى order 98 كطوق نجاة) ثم إزالة القديم من base في الإصدار التالي |
| **Q5** | كلمة مدير إجبارية؟ | توصية سارية ما لم تعترضوا: تحذير قوي + منع `none` للواي فاي بلا تأكيد صريح، دون إجبار مطلق (مرونة الميدان) |
| **Q6** | المنطقة الزمنية الافتراضية | **مؤكدة من سكربت المصنع نفسه (F6):** `Asia/Aden` + `<+03>-3` — اعتمدت في البذرة |
| **Q7** | مكان الحقول الحساسة | توصية سارية: تبويب 8 مستقل آخر الصفحة (مطابق لنموذج الرؤية المعتمد لديكم) |

---

## 12) أول خطوة تنفيذية فور الموافقة

1. إنشاء `openwrt-integration/package/luci-app-quicksetup/` في المستودع بالملفات العشرة لـP0 (بند 8).
2. كتابة `lib-discovery.sh` + `lib-build.sh` بمخرجات UCI مطابقة لمحركاتكم الحالية حرفيًا (جدول B1–B14 كمرجع اختبار).
3. `quicksetup.js` تبويبات P0 + `po/ar` كامل + بوابة grep العربية.
4. سكربت مقارنة المكافأة (اختبار 1) + دليل نشر تجريبي على WA350 بنمط أدلة DEPLOY لديكم.

*— نهاية الخطة. كل بند قابل للتنفيذ كما هو مكتوب؛ الأرقام والمهل (90s، 20s، 3 لقطات) قيم بذرة قابلة للضبط من `quicksetup` نفسه.*
