# Лаб 6: Category-Partition ба PICT

**Нэр:** Б. Аззаяа
**Код:** B242270803

## Хэсэг A: Category-Partition

### A1. Сонголт ба төлөөлөх утгууд

| Параметр | Утга | Тайлбар |
|---|---|---|
| from | Valid | данс байгаа, царцаагдаагүй |
| | Frozen | данс байгаа, царцаагдсан |
| | Missing | ийм данс байхгүй |
| to | Valid | данс байгаа, from-оос өөр |
| | Missing | ийм данс байхгүй |
| | SameAsFrom | from-той ижил данс |
| amount | Normal | 0 < дүн ≤ үлдэгдэл, өдрийн лимитэд багтана |
| | Zero | дүн ≤ 0 (0 болон сөрөг утга нэг ангид орно) |
| | OverBalance | хөрвүүлсний дараах дүн > үлдэгдэл |
| | OverLimit | дүн ≤ үлдэгдэл, гэхдээ өдрийн лимитээс хэтэрнэ |
| | ExactBalance | дүн = үлдэгдэл (хязгаарын утга) |
| currency | Same | дансны валюттай ижил |
| | Different | өөр валют, ханшаар хөрвүүлнэ |

**Далд орчны сонголт:** (1) from дансны үлдэгдэл, (2) өдрийн лимитээс үлдсэн хэмжээ, (3) ханш.
Эдгээр нь хүсэлтийн параметр биш ч үр дүнг тодорхойлдог. Жишээ нь `OverBalance` ба `OverLimit`-ийг тусад нь шалгахын тулд лимитийн хоёр өөр төлөв хэрэгтэй:
- **S1:** үлдэгдэл 1000, лимитээс үлдсэн 5000 (лимит том)
- **S2:** үлдэгдэл 1000, лимитээс үлдсэн 500 (лимит жижиг)

### A2. Хязгаарлалтууд

| Утга | Тэмдэглэгээ | Үндэслэл |
|---|---|---|
| from = Frozen | [ERROR] | ERROR_FROZEN буцаах тул бусад утгатай хослуулбал өөр алдааг нууцлана |
| from = Missing | [ERROR] | ERROR_NO_FROM буцаах тул бусад алдааг нууцлана |
| to = Missing | [ERROR] | ERROR_NO_TO буцаана |
| to = SameAsFrom | [ERROR], [IF from ≠ Missing] | ERROR_SAME_ACCOUNT буцаана; from байхгүй бол "from-той ижил" гэдэг утгагүй |
| amount = Zero | [ERROR] | ERROR_BAD_AMOUNT буцаана |
| amount = OverBalance | [ERROR] | ERROR_INSUFFICIENT буцаана |
| amount = OverLimit | [ERROR] | ERROR_LIMIT буцаана |
| amount = ExactBalance | [SINGLE] | хязгаарын утга, үр дүн нь OK; бусад параметрээс хамаарахгүй тул нэг удаа шалгахад хангалттай |
| currency = Different | [property: хөрвүүлэлттэй] | үлдэгдэл ба лимитийг хөрвүүлсний дараах дүнгээр шалгадаг |

**Давуу эрэмбэ:** from байхгүй → from царцаагдсан → to байхгүй → to = from → amount ≤ 0 → үлдэгдэл хүрэлцэхгүй → лимит хэтэрсэн.
Үндэслэл: эхлээд оролцогч дансуудын байдлыг (хямд, бүтцийн шалгалт), дараа нь дүнгийн хүчинтэй байдлыг, эцэст нь үлдэгдэл шаардсан бизнесийн шалгалтыг хийнэ. Энэ нь спецификацид дүрмүүд бичигдсэн дараалалтай нэг чиглэлтэй.

**"N → M" тооцоо:**
- Бүрэн олонлог: `3 × 3 × 5 × 2 = 90`
- [IF] хязгаарлалтаар утгагүй хослол хасна (from = Missing ∧ to = SameAsFrom: `1 × 1 × 5 × 2 = 10`): `90 → 80`
- [ERROR] утга бүрийг нэг удаа (7: Frozen, Missing, to-Missing, SameAsFrom, Zero, OverBalance, OverLimit), [SINGLE]-ийг нэг удаа (1), үлдсэн хэвийн утгуудаас (from = Valid, to = Valid, amount = Normal, currency = Same/Different) 2 хослол:
  `7 + 1 + 2 = 10` → **90 → 10**

(PICT-ийн моделд SINGLE утга ордоггүй тул тэнд бүрэн олонлог `3 × 3 × 4 × 2 = 72`. transfer.model-ийн 3 `IF` хязгаарлалтын дараа хүчинтэй хослол `72 → 40`.)

### A3. Спецификацийн хүснэгт

Тохиргоо: данс A (үлдэгдэл 1000 USD), данс B, B-ээс өөр байхгүй данс A99/B99, ханш 1 EUR = 1.1 USD.

| # | from | to | amount | currency | орчин | Хүлээгдэх үр дүн |
|---|---|---|---|---|---|---|
| 1 | Valid | Valid | Normal (100) | Same | S1 | 201 OK (happy path) |
| 2 | Valid | Valid | Normal (100) | Different (EUR) | S1 | 201 OK (110 USD хасагдана) |
| 3 | Frozen | Valid | Normal (100) | Same | S1 | 200 ERROR_FROZEN |
| 4 | Missing | Valid | Normal (100) | Same | S1 | 200 ERROR_NO_FROM |
| 5 | Valid | Missing | Normal (100) | Same | S1 | 200 ERROR_NO_TO |
| 6 | Valid | SameAsFrom | Normal (100) | Same | S1 | 200 ERROR_SAME_ACCOUNT |
| 7 | Valid | Valid | Zero (0) | Same | S1 | 200 ERROR_BAD_AMOUNT |
| 8 | Valid | Valid | OverBalance (1500) | Same | S1 | 200 ERROR_INSUFFICIENT |
| 9 | Valid | Valid | OverLimit (600) | Same | S2 | 200 ERROR_LIMIT |
| 10 | Valid | Valid | ExactBalance (1000) | Same | S1 | 201 OK (SINGLE, хязгаарын утга) |
| 11 | Frozen | Valid | Zero (0) | Same | S1 | 200 ERROR_FROZEN (давхар: from нь amount-аас өмнө) |
| 12 | Valid | Missing | OverBalance (1500) | Same | S1 | 200 ERROR_NO_TO (давхар: to нь amount-аас өмнө) |
| 13 | Valid | Valid | 1500 (үлдэгдэл ба лимит хоёулаа хэтэрнэ) | Same | S2 | 200 ERROR_INSUFFICIENT (давхар: үлдэгдэл нь лимитээс өмнө) |

Мөр 11, 12, 13 нь давхар нөхцөл бөгөөд хүлээгдэх үр дүн нь дээрх давуу эрэмбээс гарна.

## Хэсэг B: PICT

### B1. Браузерын жишээ (`browser.model`)

| Түвшин | Мөрийн тоо | Файл |
|---|---|---|
| Бүрэн олонлог (3 × 3 × 2 × 2 × 2 × 2) | 144 | - |
| 2-way (pairwise) | **10** | `results/browser-2way.txt` |
| 3-way (`/o:3`) | **20** | `results/browser-3way.txt` |

**2-way дүн:** Лабын даалгаварт лекц дээр гараар 9 мөрөнд багтаасан бол PICT 10 мөр гаргасан, өөрөөр хэлбэл 1 мөр илүү.
PICT нь greedy алгоритм ашигладаг: алхам бүрт одоогоор хамраагүй хамгийн олон хосыг шинээр хамарсан мөрийг сонгоно. Энэ арга хурдан боловч глобал хамгийн цөөн мөртэй covering array олохыг баталгаалдаггүй. Гараар бол хэв шинжийг харж, хамгийн цөөн мөрийг олж болдог.

**3-way дүн:** Бүрэн олонлог 144 мөрөөс 3-way хамралтыг 20 мөрөөр (ойролцоогоор 14%) хангасан. Хамралтын түвшинг 2-оос 3 болгоход мөрийн тоо 10-аас 20 болж 2 дахин өссөн ч бүрэн олонлогоос хамаагүй бага хэвээр байна.

README-д бичсэн мөрийн тоо нь `tail -n +2 results/browser-2way.txt | wc -l` болон `tail -n +2 results/browser-3way.txt | wc -l` командын үр дүнтэй таарна (толгойн мөрийг тооцоогүй).