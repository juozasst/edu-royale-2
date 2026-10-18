Tu esi patyręs žaidimų programuotojas. Kursime naršyklinį lavinamąjį žaidimą, panašų į Clash Royale. Dirbsime etapais: įgyvendink TIK tą etapą, kurį nurodau, parodyk rezultatą ir lauk mano patvirtinimo prieš eidamas toliau. Kodą rašyk tvarkingai, su komentarais, kad galėčiau jį suprasti ir keisti.

=== 1. PROJEKTO APRAŠYMAS ===
Realaus laiko 1v1 strategija prieš kompiuterio botą. Žaidėjas už eliksyrą deda kortas (karius, burtus, pastatus) į areną, vienetai patys eina ir kovoja, tikslas – sugriauti priešo bokštus.
Unikalu: kortų savybės tobulinamos teisingai atsakant į mokomuosius klausimus (biologija, geografija, fizika ir kt.). Žaidimas skirtas įvairioms amžiaus grupėms.

=== 2. TECHNOLOGIJOS ===
- Phaser 3 + TypeScript + Vite
- Jokio backend'o šiame etape. Progresas saugomas localStorage.
- Klausimai ir kortų statistikos – JSON failuose (ne kode!)
- Turi veikti telefone (portretinė orientacija, touch) ir kompiuteryje (pelė). Arena išlaiko proporcijas ir prisitaiko prie ekrano.
- Grafika: kol kas paprastos figūros (apskritimai, stačiakampiai) skirtingomis spalvomis + trumpas pavadinimas ant vieneto. Vėliau pakeisiu sprite'ais, todėl atvaizdavimą laikyk atskirai nuo logikos.
- Visa vartotojo sąsaja lietuvių kalba. Kodas, kintamieji, failų vardai – anglų kalba.
- Galutinis build'as turi būti statiniai failai, kuriuos galėčiau įkelti į GitHub Pages (nustatyk Vite `base: './'`).

=== 3. ARCHITEKTŪRA ===
- Simuliacija su FIKSUOTU žingsniu: 20 tick/s, atskirta nuo atvaizdavimo (render interpoliuoja). Logika turi būti deterministinė (naudok seeded random), nes vėliau bus multiplayer.
- Žaidimo logika (src/sim/) NEPRIKLAUSO nuo Phaser – tai gryni TypeScript moduliai. Phaser (src/scenes/, src/render/) tik piešia būseną ir perduoda įvestį.
- Kiekvieną tick'ą: 1) eliksyras 2) spawn/deploy 3) taikinių parinkimas 4) judėjimas + susidūrimai 5) atakos ir sviediniai 6) mirusių pašalinimas 7) pergalės tikrinimas.
- Žaidėjo veiksmas aprašomas kaip komanda: { tick, playerId, cardId, x, y }. Botas naudoja tas pačias komandas.

Failų struktūra:
src/
  main.ts
  config.ts                  // visos konstantos
  sim/  (Battle.ts, Entity.ts, Unit.ts, Tower.ts, Projectile.ts, Spell.ts, Targeting.ts, Movement.ts, Elixir.ts, Deck.ts, Bot.ts)
  scenes/ (MenuScene.ts, BattleScene.ts, CardsScene.ts, QuizScene.ts, ResultScene.ts, SettingsScene.ts)
  render/ (UnitRenderer.ts, UI.ts)
  progress/ (Progress.ts, Upgrades.ts)
  quiz/ (QuestionBank.ts, Quiz.ts)
public/data/
  cards.json
  questions/biologija.json, geografija.json, fizika.json, matematika.json, astronomija.json, chemija.json

=== 4. ARENA IR TAISYKLĖS ===
- Tinklelis 18 x 32 langelių. Žaidėjas apačioje, botas viršuje.
- Upė per vidurį (eilutės 15–16), 2 tiltai (stulpeliuose ~3 ir ~14). Žemės vienetai upę kerta tik tiltais, oro vienetai – bet kur.
- Bokštai kiekvienai pusei:
  - 2 Princess bokštai: HP 1400, žala 50, atakos intervalas 0.8 s, nuotolis 7.5 langelio, taikosi į žemę ir orą.
  - 1 King bokštas: HP 2400, žala 50, intervalas 1.0 s, nuotolis 7. Neaktyvus, kol negauna žalos arba kol nesugriaunamas savo pusės Princess bokštas.
- Padėjimo zona: tik savo pusė. Sugriovus priešo Princess bokštą, toje juostoje zona išsiplečia iki 4 eilučių į priešo pusę. Burtus galima mesti bet kur.
- Kortos padėjimas: vilk kortą iš rankos ant arenos ARBA paspausk kortą, tada vietą. Rodyk leistiną zoną ir šešėlį vietoje, kur korta atsiras.
- Matas: 3:00 min. Paskutinę 1:00 – eliksyras 2x greičiau. Lygiosios → pratęsimas 1:00 su 2x eliksyru, sudden death (pirma sugriauta bet kuri bokštas laimi). Jei vis dar lygu – laimi tas, kurio silpniausias bokštas turi daugiau HP procentų.
- Karūnos: Princess bokštas = 1, King bokštas = 3 ir iškart pergalė.

=== 5. ELIKSYRAS IR KORTOS ===
- Eliksyras: pradžioje 5, max 10, +1 kas 2.8 s (2x režime kas 1.4 s). Rodyk juostą su dalinėmis reikšmėmis.
- Kaladė 8 kortos, rankoje 4, rodoma kita korta. Panaudota korta eina į eilės galą. Pradžioje kaladė sumaišoma.
- Vieneto „deploy“ laikas 1 s (pusiau permatomas, neaktyvus).

Vieneto būsenos: DEPLOYING → WALKING → ATTACKING → DEAD.
- WALKING: juda savo juosta link artimiausio priešo pastato (per artimiausią tiltą). Juosta nustatoma pagal x poziciją.
- Taikinio parinkimas kas tick'ą tik kai neturi taikinio: artimiausias tinkamas priešas „aggro“ spindulyje (5.5 langelio). „Tik pastatai“ vienetai ignoruoja karius.
- Pradėjęs ataką, vienetas laikosi taikinio, kol šis miršta ar išeina iš nuotolio + 1 langelio.
- Susidūrimai: vienetai yra apskritimai, stumia vienas kitą (paprastas atstūmimas). Oro ir žemės vienetai tarpusavyje nesusiduria. Pastatai nejudinami.
- Ranged vienetai šauna sviediniais (matomi, skrenda iki taikinio). Splash žala – spinduliu aplink taikinį.

Pradinės 8 kortos (įrašyk į cards.json, greitis langeliais per sekundę, laikas sekundėmis):
1. Riteris – 3 eliks., HP 1450, žala 160, interv. 1.2, nuotolis 1.2, greitis 1.0, žemė, taikosi: žemė.
2. Lankininkai – 3 eliks., 2 vnt., HP 250, žala 85, interv. 1.0, nuotolis 5, greitis 1.0, žemė, taikosi: žemė+oras.
3. Milžinas – 5 eliks., HP 3300, žala 210, interv. 1.5, nuotolis 1.2, greitis 0.75, žemė, taikosi: tik pastatai.
4. Goblinai – 2 eliks., 3 vnt., HP 170, žala 100, interv. 1.1, nuotolis 0.8, greitis 1.5, žemė, taikosi: žemė.
5. Muškietininkė – 4 eliks., HP 600, žala 180, interv. 1.0, nuotolis 6, greitis 1.0, žemė, taikosi: žemė+oras.
6. Mažasis drakonas – 4 eliks., HP 800, žala 130, interv. 1.6, nuotolis 3.5, greitis 1.5, ORAS, splash 1.0, taikosi: žemė+oras.
7. Ugnies kamuolys (burtas) – 4 eliks., žala 570 (bokštams 30%), spindulys 2.5, skrydis nuo King bokšto ~1 s, nedidelis atstūmimas.
8. Patranka (pastatas) – 3 eliks., HP 800, žala 120, interv. 0.9, nuotolis 5.5, taikosi: tik žemė, gyvuoja 30 s (HP mažėja laikui bėgant).

cards.json struktūra: kiekviena korta turi id, name, type (troop|spell|building), cost, count, stats {...}, color. Visos reikšmės imamos iš JSON, kad galėčiau balansuoti be kodo keitimo.

=== 6. BOTAS ===
Paprastas, bet ne kvailas, su 3 sunkumo lygiais (Lengvas/Vidutinis/Sunkus):
- Laukia, kol sukaups eliksyro (pagal sunkumą 6–9).
- Gynyba: jei žaidėjo vienetai jo pusėje – deda tinkamą kortą prieš juos (oro vienetams – kortą, kuri taikosi į orą; būriui – splash ar burtą).
- Puolimas: kai nėra grėsmės, deda tanką (Milžiną/Riterį) prie galinės linijos ir už jo palaikymą silpnesnėje juostoje.
- Reakcijos vėlavimas: Lengvas 2 s, Vidutinis 1 s, Sunkus 0.4 s. Lengvas kartais padaro atsitiktinį ėjimą.

=== 7. LAVINAMOJI DALIS: KORTŲ TOBULINIMAS ===
Kiekviena korta turi savybes, kiekviena susieta su mokslo sritimi:
- Gyvybės (HP) → biologija
- Judėjimo greitis → geografija
- Žala → fizika
- Atakos greitis → matematika
- Nuotolis → astronomija (tik ranged kortoms)
- Burto žala / spindulys → chemija (tik burtams)

Taisyklės:
- Kiekviena savybė turi lygius 0–6. Kiekvienas lygis +5% (max +30%). Atakos greičiui – intervalas mažėja 5% per lygį.
- Norint pakelti lygį reikia atsakyti teisingai iš eilės: lygiai 1–2 → 1 klausimas, 3–4 → 2 klausimai, 5–6 → 3 klausimai.
- Klausimo sunkumas (difficulty 1–3) = pagal lygį: 1–2 → 1, 3–4 → 2, 5–6 → 3.
- Neteisingas atsakymas: rodomas teisingas atsakymas + paaiškinimas, ta korta+savybė užrakinama 1 kovai (rodyk spyną ir užrašą „Atsirakins po kitos kovos“).
- Teisingas atsakymas taip pat rodo paaiškinimą (trumpai).
- Variantų tvarka kaskart maišoma.
- Neteisingai atsakyti klausimai įrašomi į „pakartojimų“ sąrašą ir vėliau rodomi su didesne tikimybe (paprastas spaced repetition). Ką tik teisingai atsakytas klausimas nekartojamas bent 20 klausimų.
- Jei klausimų pagal filtrą nebeliko – imami lengvesni arba kartojami seniausi.

Ekranas „Kortos“ (CardsScene): visos 8 kortos, paspaudus – kortos detalės su kiekviena savybe, jos lygiu, sritimi (ikona/spalva), mygtuku „Tobulinti“. Rodyk, kiek ši savybė pakeistų skaičių (pvz. HP 1450 → 1522).
Kaladė kol kas fiksuota (tos 8 kortos), redaktorius bus vėliau.

=== 8. AMŽIAUS GRUPĖS IR KLAUSIMAI ===
Pirmo paleidimo metu žaidėjas įveda vardą ir pasirenka amžiaus grupę (vėliau galima keisti nustatymuose):
- "1-4" (1–4 klasė), "5-8" (5–8 klasė), "9-12" (9–12 klasė), "adult" (suaugę)

Klausimo formatas:
{
  "id": "bio_014",
  "subject": "biologija",
  "ageGroup": "5-8",
  "difficulty": 2,
  "question": "Kuris organas pumpuoja kraują?",
  "answers": ["Širdis", "Plaučiai", "Kepenys", "Inkstai"],
  "correct": 0,
  "explanation": "Širdis yra raumuo, kuris susitraukdamas varo kraują kraujagyslėmis."
}
- Visada 4 atsakymų variantai. Faktai turi būti teisingi ir tinkami amžiui.
- QuestionBank turi patikrinti JSON (validacija: 4 atsakymai, correct 0–3, unikalūs id) ir konsolėje įspėti apie klaidas.
- Sugeneruok pradinį rinkinį: kiekvienai sričiai ir kiekvienai amžiaus grupei bent po 6 klausimus (po 2 kiekvieno sunkumo). Lietuvių kalba.

=== 9. PROGRESAS ===
localStorage saugo: vardą, amžiaus grupę, kortų savybių lygius, užrakinimus, pakartojimų sąrašą, statistiką (kovos, pergalės, karūnos, teisingi/neteisingi atsakymai pagal sritį). Versijuok saugojimo formatą (saveVersion), kad vėliau galėčiau migruoti. Nustatymuose – mygtukas „Ištrinti progresą“ su patvirtinimu.

=== 10. EKRANAI ===
- Meniu: „Kovoti“ (pasirinkti boto sunkumą), „Kortos“, „Statistika“, „Nustatymai“.
- Kova: arena, laikmatis, karūnos, eliksyro juosta, 4 kortos + kita korta, HP juostos virš vienetų ir bokštų, žalos skaičiai (galima išjungti), pauzės mygtukas (tik prieš botą).
- Rezultatas: pergalė/pralaimėjimas, karūnos, mygtukai „Dar kartą“ ir „Į meniu“. Po kovos atrakinami užrakinti patobulinimai.

=== 11. ETAPAI (daryk po vieną!) ===
1. Projekto karkasas (Vite + Phaser + TS), arena su upe, tiltais, 6 bokštais, kurie šaudo į bandomąjį vienetą. Fiksuoto žingsnio ciklas.
2. Eliksyras, kaladė, ranka, kortų padėjimas (drag ir tap), padėjimo zonos.
3. Vienetai pagal cards.json: būsenų mašina, judėjimas per tiltus, taikiniai, atakos, sviediniai, splash, susidūrimai, burtas, pastatas.
4. Laikmatis, 2x eliksyras, pratęsimas, karūnos, pergalė, rezultato ekranas.
5. Botas su 3 sunkumais.
6. Meniu, progresas localStorage, pirmo paleidimo ekranas (vardas + amžiaus grupė).
7. Klausimų bankas, validacija, pradiniai klausimai.
8. Kortų ekranas ir tobulinimas per klausimus (QuizScene), užrakinimai, pakartojimai. Patobulinimai taikomi kovoje.
9. Statistika, nustatymai, poliravimas (animacijos, garso vietos, mobilus pritaikymas), build'as GitHub Pages.
