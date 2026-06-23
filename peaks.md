# UA Delivery Tickets — analýza peakov a trendov

LLM klasifikácia 4 448 customer-initiated UA ticketov o doručení (2025-05 → 2026-06). Z pôvodných 8 336 ticketov boli odfiltrované:
- 2 242 mis-classified ako delivery (skutočne o vrátiach/platbách/iných témach)
- 1 624 z `data_or_system_error` ktoré neovplyvňujú doručenie (payment fails, promo kódy, duplikáty, account issues) — ponechané len problémy s **adresou/kontaktom**
- 31 ticketov pred 2025-05 (neúplné backfill data)

Pre každú kategóriu nájdeš trend, príčinu, ukážkové tickety s Ladesk linkami a popisom typickej situácie.

---

## 1. `address_or_contact_issue` — STABILNÝ ~10-29/mes

### Trend
| Obdobie | Priemer/mes |
|---|---|
| Máj-Dec 2025 | 12-19 |
| Jan-Apr 2026 | 20-29 (mierny ramp) |
| Máj-Jún 2026 | 9-19 |

Nie je tu dramatický peak. Mierny nárast od jan 2026 (možno korelácia s customs ramp-om — pri nadrozmernom prísnejšom colnictve zákazníci si všímajú detaily adresy).

### Príčina
Zákazník alebo agent zistil, že **adresa/telefón/meno príjemcu** v objednávke nie sú správne, a treba ich opraviť pred / počas doručenia.

### Vzorové tickety
- 🎫 [CLD-RGFWJ-358](https://gymbeam.ladesk.com/agent/index.php?rnd=8325#Conversation;id=p8nwht0w) — *2025-11-07* — resol: `info_only`, urgency: `low`
  > Zákazník nahlásil, že platbu vykonal, ale príjemcom objednávky nie je on, ale iná osoba s konkrétnym menom a telefónnym číslom. Agent potvrdil zmenu údajov príjemcu a požiadal o zaslanie potvrdenia o platbe vo formáte PDF pre rýchlejšie spracovanie finančným oddelením.
- 🎫 [RQS-KCHHQ-043](https://gymbeam.ladesk.com/agent/index.php?rnd=8325#Conversation;id=kj58vpa4) — *2026-06-11* — resol: `escalated_to_carrier`, urgency: `low`
  > Zákazník nahlásil, že mu neukazuje zásielku v aplikácii. Agent zistil, že zákazník zadal chybné telefónne číslo. Agent požiadal o správne číslo a následne požiadal prepravcu o zmenu kontaktných údajov.
- 🎫 [DPS-JKHPG-157](https://gymbeam.ladesk.com/agent/index.php?rnd=8325#Conversation;id=i8bac17p) — *2025-07-21* — resol: `info_only`, urgency: `low`
  > Zákazník má problém s vyplnením adresy pre doručenie na pobočku Novej pošty v Kyjeve, konkrétne s pobočkou č. 90. Agent mu radí, ako správne zadať údaje do formulára: najprv mesto Kyjev, potom číslo pobočky 90, a následne ulicu a PSČ.

### Akcia
- Validácia adresy už pri checkoute (autocomplete z Nova Post adresára obce/ulice)
- Povinné overenie telefónneho čísla pred odoslaním (možno SMS verifikácia)

---

## 2. `failed_delivery_customer_side` — RAMP od Dec 2025 (3-5×)

### Trend
| Obdobie | Priemer/mes |
|---|---|
| Máj-Nov 2025 | 3-4 |
| **Dec 2025+** | **9-18** |

### Príčina
Dominantne: **balík vrátený, lebo vypršal storage period v Nova Post pobočke** (*"посилку було повернуто, так як минув термін зберігання"*). Plus false "delivered" notifikácie a refusal-to-collect.

### Vzorové tickety
- 🎫 [QLB-LMLBN-729](https://gymbeam.ladesk.com/agent/index.php?rnd=8325#Conversation;id=ah4cgvyj) — *2026-05-10* — resol: `info_only`, urgency: `medium`
  > Zákazníčka sa informovala o stave svojej objednávky. Agent zistil, že objednávka sa vrátila späť, pretože uplynula doba jej uskladnenia na pobočke.
- 🎫 [BFN-ZMDRK-448](https://gymbeam.ladesk.com/agent/index.php?rnd=8325#Conversation;id=3vy7pnat) — *2026-04-16* — resol: `refund_full`, urgency: `low`
  > Zákazník odmietol prevziať zásielku a informoval o tom GymBeam. Agent potvrdil, že peniaze budú vrátené do 3-5 pracovných dní.
- 🎫 [LBC-VDPWM-897](https://gymbeam.ladesk.com/agent/index.php?rnd=8325#Conversation;id=3cape1pg) — *2026-06-17* — resol: `order_cancelled`, urgency: `low`
  > Zákazník kontaktoval zákaznícke centrum s tým, že neočakáva žiadnu zásielku z internetového obchodu GymBeam, hoci mu bolo doručené oznámenie o objednávke SK0000000166325664NPG/20700707966497. Agent GymBeam požiadal o vrátenie tejto objednávky, čím sa problém vyriešil.

### Akcia
Preveriť Nova Post UA storage policy zmeny od Q4 2025; upraviť GymBeam upozornenia pred vypršaním lehoty.

---

## 3. `carrier_other_problem` — JAN 2026 SPIKE (34 vs 1-4 inokedy)

### Trend
| Mesiac | Počet |
|---|---|
| Predtým / potom | 1-4 |
| **2026-01** | **34** |

### Príčina
**Konkrétny incident 5.1.2026** — Nova Post integration failure pre cash-on-delivery. QR kódy na pobočkách nešli zoskenovať, suma sa zobrazovala v eurách namiesto hrivien, zákazníci nevedeli zaplatiť.

V 34 ticketoch agenti posielali identickú odpoveď: *"Pre objednávky od 05.01 s dobierkou vznikla technická chyba pri prenose dát medzi medzinárodnou dopravou a doručením v rámci Ukrajiny."*

### Vzorové tickety
- 🎫 [MVM-GMXCT-723](https://gymbeam.ladesk.com/agent/index.php?rnd=8325#Conversation;id=605j4aj7) — *2026-01-10* — resol: `info_only`, urgency: `medium`
  > Zákazník sa pýtal na možnosti platby za objednávku, keďže nemohol zaplatiť na pošte ani cez aplikáciu Nova Pošta, kde sa mu zobrazovala platba v eurách namiesto hrivien. Agent informoval, že išlo o technickú chybu pri prenose dát medzi medzinárodnou a ukrajinskou doručovacou službou, ktorá už bola odstránená. Zásielky budú presmerované do Kyjeva na preetiketovanie a následne opätovne odoslané zákazníkom.
- 🎫 [CSW-JRSVD-781](https://gymbeam.ladesk.com/agent/index.php?rnd=8325#Conversation;id=t3z44gf6) — *2026-01-10* — resol: `info_only`, urgency: `medium`
  > Zákazník mal problém s platbou za objednávku na dobierku, keďže sa mu nepodarilo naskenovať QR kód pre platbu. Agent informoval zákazníka, že platba je možná cez aplikáciu Novej Pošty alebo priamo na pobočke a že spôsob platby už nie je možné zmeniť, keďže objednávka opustila sklad.
- 🎫 [SHQ-VZKZC-154](https://gymbeam.ladesk.com/agent/index.php?rnd=8325#Conversation;id=i0jvnb7i) — *2026-01-11* — resol: `escalated_to_carrier`, urgency: `medium`
  > Zákazník nemohol zaplatiť dobierku za objednávku, pretože v aplikácii Nova Post sa mu zobrazovala suma v eurách namiesto hrivien a platba cez terminál nebola možná. Agent informoval, že išlo o technickú chybu pri prenose dát medzi medzinárodnou a ukrajinskou doručovacou službou, ktorá už bola odstránená. Všetky dotknuté zásielky budú presmerované do Kyjeva na preetiketovanie a následne opätovne odoslané zákazníkom.

### Akcia
Pridať alerting na anomálne ticket volumes per problem_type — chytí budúce podobné incidenty v deň 1, nie v deň 30.

---

## 4. `customs_or_border_held` — RAMP od Jan 2026 (5-6× a držia sa)

### Trend
| Obdobie | Priemer/mes |
|---|---|
| Máj-Nov 2025 | 1-5 |
| **Jan 2026+** | **17-29 (sustained)** |

### Príčina
**Zmena UA cross-border pravidiel okolo dec 2025 / jan 2026:**
- Objednávky **nad 150 €** sa musia precolniť (zákazník platí customs poplatky)
- Obmedzenia na **potravinové produkty** v medzinárodnej preprave

Systémový (nie incident) trend — corresponds s posilnením vymáhania UA personal import duty pravidiel.

### Vzorové tickety
- 🎫 [XLH-SPNMP-632](https://gymbeam.ladesk.com/agent/index.php?rnd=8325#Conversation;id=mq7zyog9) — *2026-04-17* — resol: `info_only`, urgency: `low`
  > Zákazník sa informoval o stave svojej objednávky, ktorá ešte nebola doručená. Agent zistil, že zásielka je už na Ukrajine a prechádza colným konaním, pričom doručenie sa očakáva do 1-2 dní. Meškanie je spôsobené vysokým počtom objednávok v dôsledku výpredaja.
- 🎫 [CMN-GRNBX-521](https://gymbeam.ladesk.com/agent/index.php?rnd=8325#Conversation;id=01e6oikv) — *2026-03-30* — resol: `info_only`, urgency: `medium`
  > Zákazník neobdržal objednávku, ktorá je stále na colnici, hoci systém hlási doručenie. Problém spočíva v tom, že hodnota objednávky presahuje 150 eur, čo si vyžaduje colné odbavenie a úhradu cla a DPH podľa ukrajinských zákonov. Agent vysvetlil zákazníkovi proces colného odbavenia a nutnosť úhrady poplatkov, ktoré sú povinné pre medzinárodné zásielky nad stanovený limit.
- 🎫 [THK-JRXCW-754](https://gymbeam.ladesk.com/agent/index.php?rnd=8325#Conversation;id=avw5q5q9) — *2026-05-06* — resol: `reorder_sent`, urgency: `medium`
  > Zákazník nahlásil, že jeho objednávka 25003592960 nebola doručená a je zablokovaná na colnici. Nežiadal vrátenie peňazí, ale opätovné doručenie tovaru. Agent zabezpečil opätovné odoslanie objednávky s novým číslom 25003616918 a doručením na 9.-10. mája.

### Akcie
- Pridať warning na checkoute pre objednávky nad 150 € (ukázať očakávané customs poplatky)
- Rozdeliť veľké objednávky na menšie pod threshold, kde je to možné
- Update FAQ / customer education o customs procese

---

## 5. `lost_in_transit` — RAMP Q1-Q2 2026 (3-4×)

### Trend
| Obdobie | Priemer/mes |
|---|---|
| Máj-Sep 2025 | 1-5 |
| Q1-Q2 2026 | 9-17 (peak Máj 2026: 17) |

### Príčiny (tri prepojené)

**a) Nova Post Europe batch reporty o stratených zásielkach**
Emaily od **Ilony Pediura (Care Centre Slovakia/Hungary, Nova Post Europe)** so zoznamami chýbajúcich SHSK trackingov. Jeden ticket vie obsahovať 10+ stratených SHSK.

**b) Vojnové škody na infraštruktúre Nova Post**
Konkrétne: **apríl 2026 — útok na terminál Nova Post v Lucku.** Sklad zhorel, zásielky zničené.

**c) Politika riešenia sa zmenila — refund namiesto reorder**
| Obdobie | reorder_sent | refund_full |
|---|---|---|
| do 2025-12 | 68 % | 12 % |
| 2026-03+ | 23 % | **37 %** |

Agenti v 2026 píšu *"ми не можемо повторно надіслати повернену посилку"* — defaultujú na refund.

### Vzorové tickety
- 🎫 [BHN-ZRJMT-445](https://gymbeam.ladesk.com/agent/index.php?rnd=8325#Conversation;id=uz89e3dq) — *2026-04-07* — resol: `reorder_sent`, urgency: `medium`
  > Zákazníčka sa informovala na číslo zásielky, pretože nedostala svoju objednávku. Agent zistil, že objednávka bola pravdepodobne stratená počas prepravy. Zákazníčke bola ponúknutá možnosť opätovného odoslania, vrátenia peňazí alebo kupónu, pričom zákazníčka si vybrala opätovné odoslanie.
- 🎫 [XQJ-HCDFV-575](https://gymbeam.ladesk.com/agent/index.php?rnd=8325#Conversation;id=g9rv7zjm) — *2026-03-28* — resol: `refund_full`, urgency: `medium`
  > Zákazník nahlásil, že jeho zaplatená objednávka bola zrušená. Agent potvrdil, že objednávka bola stratená počas prepravy a peniaze budú automaticky vrátené.
- 🎫 [RZX-VLMTF-818](https://gymbeam.ladesk.com/agent/index.php?rnd=8325#Conversation;id=ty0a8l41) — *2026-06-15* — resol: `escalated_to_carrier`, urgency: `medium`
  > Zákazník, zástupca spoločnosti Nova Post Europe, žiadal o potvrdenie, ktoré zásielky môže vymazať zo systému. Agent potvrdil, že všetky uvedené zásielky môžu byť vymazané.

### Akcie
- Monitorovať Iloniné batch reporty automaticky (loss-rate per shipment batch)
- Pre objednávky nad 150 € zvážiť poistenie alebo dôraz na balenie
- Re-evaluovať refund-vs-reorder policy z pohľadu retention

---

## Celkový obraz

Štyri trendy s rampom od Dec 2025 / Jan 2026 — `customs_or_border_held`, `lost_in_transit`, `failed_delivery_customer_side`, mierne aj `address_or_contact_issue` — vyzerajú ako **kumulatívny dôsledok zhoršenia UA logistickej situácie**:
- Sprísnené UA colné kontroly (od ~Q4 2025)
- Útoky na Nova Post infraštruktúru
- Skrátený storage window (hypotéza)
- Zmena GymBeam interných policies smerom k refund-only

**Poznámka k pôvodnej analýze `data_or_system_error`:** v predošlej verzii sme videli peak Sep-Nov 2025 (310 ticketov), ktorý sa ukázal byť **payment problémami** (85-90 % obsahu). Tieto sú teraz odfiltrované, lebo netýkajú sa doručenia. Trend v platbách bol vyriešený okolo dec 2025 — payment integration sa pravdepodobne fixla.

---

## Priorizované akcie

| Priorita | Akcia |
|---|---|
| **Vysoká** | Customs transparency na checkoute pre objednávky nad 150 € |
| **Vysoká** | Validácia adresy a telefónu pri checkoute (autocomplete + verifikácia) |
| Stredná | Automatizovať monitoring Nova Post Europe batch reportov |
| Stredná | Preveriť Nova Post UA storage policy a upraviť customer notifikácie pred expiry |
| Stredná | Re-evaluovať refund-vs-reorder policy |
| Nízka | Pridať alerting na anomálne ticket volumes per problem_type |
