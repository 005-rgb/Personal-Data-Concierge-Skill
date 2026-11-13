
# Global Reference Data

General guidance for the `personal-data-concierge` Skill. Not official, may be outdated, and never overrides the user's own checklist or the rules of the issuing agency. The agents cannot read this file: copy the keywords or rules they need into each delegation instruction.

Sections: G1 keywords by category · G2 calendars and eras · G3 impact classes · G4 procedure templates · G5 date-order conventions.

## G1. Multilingual keywords by category

Use 3 to 8 keywords per search: the user's language, the likely document language(s), and English. Acronyms are allowed only where listed here.

Languages: EN English · ID Indonesian · JA Japanese · ZH Chinese (traditional and simplified) · TH Thai · DE German · FR French · ES Spanish · KO Korean.

### Identity and travel
| Item | Keywords |
|---|---|
| Passport | EN passport · ID paspor · JA パスポート, 旅券 · ZH 護照, 护照 · TH หนังสือเดินทาง · DE Reisepass · FR passeport · ES pasaporte · KO 여권 |
| Visa | EN visa · ID visa · JA ビザ, 査証 · ZH 簽證, 签证 · DE Visum · FR visa · ES visado · KO 비자 |
| National ID / residence card | EN national ID, residence card, residence permit · ID KTP, kartu identitas, KITAS · JA マイナンバーカード, 在留カード · ZH 身分證, 居留證, 居留证 · DE Personalausweis, Aufenthaltstitel · FR carte d'identité, titre de séjour · ES DNI, cédula, NIE · KO 주민등록증, 외국인등록증 |
| Birth / family record | EN birth certificate, family card · ID akta lahir, KK · JA 出生証明書, 戸籍 · ZH 出生證明, 戶口名簿 · DE Geburtsurkunde · FR acte de naissance · ES partida de nacimiento · KO 출생증명서 |

### Vehicle and driving
| Item | Keywords |
|---|---|
| Driving licence | EN driving licence, driver's license · ID SIM, surat izin mengemudi · JA 運転免許証 · ZH 駕照, 驾照, 駕駛執照 · TH ใบขับขี่ · DE Führerschein · FR permis de conduire · ES licencia de conducir · KO 운전면허증 |
| Vehicle registration | EN vehicle registration · ID STNK, BPKB · JA 車検証 · ZH 行照, 行驶证 · DE Fahrzeugschein, Zulassungsbescheinigung · FR carte grise · ES permiso de circulación |
| Car insurance | EN car insurance · ID asuransi mobil · JA 自動車保険 · ZH 汽車保險, 汽车保险 · DE Kfz-Versicherung · FR assurance auto · ES seguro de coche |

### Insurance
EN insurance policy · ID polis asuransi · JA 保険証券 · ZH 保單, 保险单 · TH กรมธรรม์ · DE Versicherungsschein, Police · FR police d'assurance · ES póliza de seguro · KO 보험증권

### Tax and finance
EN tax return, tax assessment · ID pajak, SPT, NPWP · JA 確定申告, 納税通知書 · ZH 稅單, 报税 · DE Steuerbescheid · FR avis d'imposition · ES declaración de la renta · KO 세금, 납세

### Property, rental and employment
| Item | Keywords |
|---|---|
| Lease / rental | EN lease, tenancy agreement · ID perjanjian sewa · JA 賃貸借契約書 · ZH 租約 · DE Mietvertrag · FR bail · ES contrato de alquiler |
| Property title | EN title deed · ID sertifikat tanah, SHM · JA 登記簿謄本 · DE Grundbuchauszug · FR titre de propriété · ES escritura |
| Employment | EN employment contract, work permit · ID kontrak kerja · JA 雇用契約書 · ZH 勞動契約, 劳动合同 · DE Arbeitsvertrag, Arbeitserlaubnis · FR contrat de travail · ES contrato de trabajo |

### Health
EN vaccination certificate, health insurance card, prescription · ID kartu vaksin, BPJS, resep · JA 予防接種証明書, 健康保険証 · ZH 疫苗證明, 健保卡 · DE Impfpass, Krankenversicherungskarte · FR carnet de vaccination, carte vitale · ES cartilla de vacunación, tarjeta sanitaria

### Education and certificates
EN diploma, degree certificate, transcript, certificate · ID ijazah, transkrip, sertifikat · JA 卒業証明書, 成績証明書 · ZH 畢業證書, 毕业证书, 成績單 · DE Zeugnis, Urkunde · FR diplôme, relevé de notes · ES título, certificado

### Warranties and receipts
EN warranty, receipt, invoice · ID garansi, kwitansi, nota, faktur · JA 保証書, 領収書, 請求書 · ZH 保固, 保修, 發票, 发票 · DE Garantie, Quittung, Rechnung · FR garantie, reçu, facture · ES garantía, recibo, factura

### Subscriptions and memberships
EN subscription, membership · ID langganan, keanggotaan · JA 会員, サブスクリプション · ZH 會員, 订阅 · DE Abonnement, Mitgliedschaft · FR abonnement, adhésion · ES suscripción, membresía

## G2. Calendars and eras

### Exact-rule conversions (deterministic)

| Calendar | Rule | Example |
|---|---|---|
| Buddhist Era (BE, Thailand) | Gregorian year = BE year − 543 | 2570 BE → 2027 |
| Minguo (ROC, Taiwan) | Gregorian year = Minguo year + 1911 | 116 → 2027 |
| Japanese era: Reiwa (令和) | Gregorian year = era year + 2018. Era starts 1 May 2019 | Reiwa 9 → 2027 |
| Japanese era: Heisei (平成) | Gregorian year = era year + 1988. 8 Jan 1989 to 30 Apr 2019 | Heisei 30 → 2018 |
| Japanese era: Showa (昭和) | Gregorian year = era year + 1925. 25 Dec 1926 to 7 Jan 1989 | Showa 60 → 1985 |
| Japanese era: Taisho (大正) | Gregorian year = era year + 1911. 1912 to 1926 | Taisho 10 → 1921 |
| Japanese era: Meiji (明治) | Gregorian year = era year + 1867. 1868 to 1912 | Meiji 40 → 1907 |

Notes:
- Keep month and day unchanged; only the year changes.
- Era year 1 is written 元年 (gannen). Near an era boundary (for example January to April 2019, or 1 to 7 January 1989), check the exact date against the era start and end dates above; if the document gives only a year at a boundary, mark `ambiguous`.
- Show both forms: `2570 BE (2027)`, `Reiwa 9 (2027)`, `Minguo 116 (2027)`.
- A Gregorian year written on the document always wins over a conversion; if the two disagree, show both and mark `low confidence`.

### Not converted (do not compute from memory)

Hijri (Islamic), Persian (Solar Hijri), Hebrew, Ethiopian, Nepali (Bikram Sambat), Indian national calendar, and any other calendar. Procedure: use a Gregorian date printed on the document; if none, show the date as written, mark `calendar: not converted`, exclude it from risk ranking, and ask the user to confirm the Gregorian date.

## G3. Impact classes by document type

Used in the risk score (High 40, Medium 25, Low 10). If the type is unclear, use Medium and flag `type uncertain`.

| Class | Document types |
|---|---|
| **High** | Passport · national ID · residence permit or card · visa · work permit · driving licence · vehicle registration · health insurance · car insurance · home or life insurance · tax assessment or filing notices · property title · lease or tenancy agreement · employment contract |
| **Medium** | Vaccination record with a booster or renewal date · professional licence or certification · utility or service contracts · loan or financing agreements · warranties for high-value items (vehicles, appliances, electronics) · school or course enrolment documents with renewal dates |
| **Low** | Receipts and invoices · consumer product warranties · subscriptions and memberships · loyalty cards · routine bills |

## G4. Procedure templates (general guidance; confirm with the issuing agency)

Requirements vary by country and agency. These templates only help to start a checklist. Always state "general guidance; confirm with the issuing agency", ask the user to add or remove items, and ask for the jurisdiction. Items marked P cannot be verified from files and get status `PHYSICAL / USER TO CONFIRM`.

### Driving licence renewal
1. Current driving licence (copy or scan)
2. National ID or residence card
3. Proof of address, if the agency requires it
4. Recent photo meeting agency specifications (P)
5. Medical or eyesight certificate, if required (P if a new one must be obtained)
6. Application form, signed (P)
7. Payment of fees (P)

### Passport renewal
1. Current passport (copy or scan), and the original at submission (P)
2. National ID or residence card
3. Birth certificate or family record, if required
4. Recent photo meeting agency specifications (P)
5. Application form, signed (P)
6. Biometrics appointment, if required (P)
7. Payment of fees (P)

### Residence permit extension
1. Current passport with remaining validity
2. Current residence permit or card
3. Proof of purpose of stay (employment contract, enrolment, or family documents, as applicable)
4. Proof of address, such as a lease
5. Proof of income or financial means, if required
6. Health insurance, if required
7. Official translations, apostille or legalization, if required (P)
8. Photo, signed forms, biometrics, fees (P)

### Vehicle registration renewal
1. Current vehicle registration document
2. Valid vehicle insurance
3. Owner's ID
4. Roadworthiness or inspection certificate, if required (P if a new one must be obtained)
5. Proof of tax or fee payment, if required
6. Signed application, fees (P)

### Annual tax filing preparation
1. Prior-year tax return or assessment
2. Income statements (employer, bank, or platform)
3. Proof of deductible expenses or allowances, as applicable
4. Insurance premium or contribution statements, if relevant
5. Tax ID (masked in any output)
6. Signed declaration or submission confirmation (P)

## G5. Date-order conventions (guidance for ambiguity checks)

Use only as supporting evidence together with the issuing country; never as the sole proof for an ambiguous date such as `04/05/2027`.

| Typical order | Common in |
|---|---|
| Year-Month-Day | Japan, China, Taiwan, Korea; ISO 8601 everywhere |
| Day-Month-Year | Indonesia, Thailand, most of Europe, the UK, Australia, most of Latin America |
| Month-Day-Year | United States, and some Philippine and Canadian documents |

Rules: a day or month field above 12 settles the order; a written month name settles it; ISO `YYYY-MM-DD` settles it. If still unclear, show the date as written and mark `ambiguous`.
