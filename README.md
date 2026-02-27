# 🎓 Schoolkeuze Simulator — Amsterdam VWO 2026

Deze app simuleert de Amsterdamse loting- en matchingprocedure voor VWO-scholen, zodat je kunt zien wat je kansen zijn op plaatsing bij elke school op je voorkeurslijst, afhankelijk van je lotnummer.

## Context: het DA-STD algoritme

De Amsterdamse loting & matching werkt met het **DA-STD algoritme** (Deferred Acceptance — Single Tie-breaking):

1. Alle ~2400 VWO-leerlingen krijgen een **willekeurig lotnummer** (1 t/m N)
2. Het algoritme verwerkt leerlingen op volgorde van lotnummer (laag = gunstig)
3. Elke leerling wordt geplaatst bij de **hoogste voorkeur** waar nog plek is
4. Als een school vol is → door naar de volgende voorkeur op de lijst
5. Leerlingen die niet op hun 1e keuze komen, worden op de **reservelijst** geplaatst

**Kernfeiten (2025):**
- 2407 VWO-leerlingen, 41 VWO-afdelingen
- 2525 totale VWO-capaciteit (ruim genoeg voor iedereen)
- 11 scholen zijn overvraagd (meer 1e-keuze dan plekken)
- 70% van de leerlingen komt op hun 1e keuze
- Voorkeurslijst: 12 scholen (voor 2026)
- Plaatsingsgarantie bij volledige lijst (capaciteit wordt zonodig +4% verhoogd)

**Bron:** [Loting-en-Matching-2025-Verslag (OSVO/Mapping Worlds)](https://www.osvo.nl/loting-en-matching)

## Hoe werkt de simulatie?

### Wat proberen we te berekenen?

We willen weten: **als jij lotnummer X krijgt, op welke school kom je dan terecht?** Omdat we jouw lotnummer van tevoren niet weten, berekenen we de kans voor elk mogelijk lotnummer (1 tot 2407).

### Stap 1: Andere leerlingen nabootsen

In werkelijkheid hebben alle 2407 leerlingen een eigen gerangschikte lijst van scholen ingediend. Die lijsten hebben we niet. We maken ze na op basis van wat we wél weten:

- **1e keuze**: we weten van elke school hoeveel leerlingen die als eerste keuze hadden (bijv. 211 leerlingen kozen St. Nicolaaslyceum). Daarmee maken we een kansenverdeling: een willekeurige leerling kiest St. Nicolaaslyceum als eerste keuze met kans 211/2347.
- **2e keuze**: idem, gebaseerd op de werkelijke tweede-keuze aantallen per school — maar altijd anders dan de eerste keuze.
- **3e keuze**: zelfde aanpak.
- **4e tot 12e keuze**: hier hebben we geen data meer. We gebruiken **clusters** (zie hieronder).

### Stap 1b: Clusters — scholen van hetzelfde type

Leerlingen zetten niet willekeurig scholen op positie 4 tot 12. Wie Barlaeus Gymnasium als eerste keuze heeft, zet waarschijnlijk ook andere gymnasiums hoog op zijn lijst — niet ineens een Montessori school. De vraag is: welke scholen lijken op elkaar vanuit het perspectief van ouders en leerlingen?

We hebben de 41 scholen ingedeeld in vijf groepen op basis van onderwijstype en schoolcultuur:

- **Gymnasium** (7): Barlaeus, Cygnus, Vossius, Ignatiusgymnasium, Het 4e Gymnasium, Spinoza Gymnasium, Montessori Gymnasium — klassieke vorming met Latijn/Grieks
- **Traditioneel lyceum** (17): Het Amsterdams Lyceum, Fons Vitae, Hyperion, Spinoza, Calandlyceum, Ir. Lely, St. Nicolaaslyceum (+tto), Hervormd Zuid/West, Pieter Nieuwland (+Plus), Damstede, Gerrit van der Veen, Comenius, Berlage-tto, Cartesius Het Lyceum — gevestigde lycea met een klassiek profiel
- **Montessori** (5): Montessori Amsterdam, Pax, Terra Nova, Metis Technasium, Metis Kunst & Co — scholen met de Montessori-pedagogiek
- **Vernieuwend** (11): Alasca, Cartesius De Plaats, DENISE, Geert Groote, Kairos, Lumion, Marcanti, Metropolis, OSB, Vinse, Xplore — nieuwere scholen met een eigen onderwijsconcept
- **Rest** (1): Cornelius Haga — past bij geen enkel ander cluster

**Waarom geen levensbeschouwelijk cluster?** Hoewel er zowel christelijke (Hervormd, Geert Groote) als islamitische (Cornelius Haga) scholen zijn, kiezen ouders die specifiek islamitisch onderwijs willen niet voor een christelijke school en vice versa. Daarom zijn deze scholen ingedeeld bij het cluster dat past bij hun *type onderwijs*, niet bij hun denominatie.

**Hoe werkt de boost?** Voor posities 4 tot 12 geven we scholen uit hetzelfde cluster een **drie keer hogere kans** om op de lijst te verschijnen dan scholen uit een ander cluster. Zo bootsen we na dat leerlingen een consistente voorkeur hebben voor een bepaald type school. **Uitzondering:** het rest-cluster krijgt géén boost (factor 1×), omdat er geen verwantschap is met andere scholen.

### Stap 2: De loting uitvoeren

Nu simuleren we het echte algoritme:

1. We geven alle 2407 leerlingen een willekeurig lotnummer.
2. We beginnen bij lotnummer 1 en gaan omhoog.
3. Elke leerling krijgt de hoogste school op zijn lijst waar nog een plek vrij is.
4. Is een school vol? Dan schuift die leerling door naar zijn volgende keuze.
5. We houden bij: als **jij** nu aan de beurt was, op welke school zou je terechtkomen?

Dit doen we voor elk lotnummer van 1 tot 2407 in één run.

### Stap 3: Extra rem op de cascade

Een belangrijk probleem ontdekten we: sommige scholen zoals St. Nicolaaslyceum hebben in werkelijkheid **alle 58 plekken gevuld via eerste keuze** — niemand komt daar binnen als tweede of derde keuze. Maar in onze gesimuleerde lijsten konden leerlingen die school ook als tweede of derde keuze hebben, waardoor die plekken te vroeg weglekten.

De oplossing: we gebruiken de werkelijke data om een **harde grens** in te stellen. Per school weten we hoeveel plekken er werkelijk via de eerste, tweede, derde en latere keuzes zijn gevuld. We laten de simulatie nooit meer plekken via een bepaalde positie vullen dan in werkelijkheid is gebeurd.

### Stap 4: Herhalen voor zekerheid

Eén simulatie geeft al een antwoord, maar door toeval kan dat soms afwijken. Daarom herhalen we het hele proces **500 keer**. Uiteindelijk middelen we de uitkomsten: als je in 400 van de 500 simulaties op Fons Vitae belandt, is de kans 80%.

## Technische details

### Gumbel-max trick voor snelle voorkeurlijst-generatie

We moeten voor elke simulatie voorkeurlijsten genereren voor ~2400 leerlingen. Individueel samplen is te langzaam. De **Gumbel-max trick** lost dit op:

- `argmax(log(p) + Gumbel_noise)` is equivalent aan `random.choice(p=p)`
- Maar werkt op hele matrices tegelijk → 1000x sneller
- Bovendien: `argsort(-scores)` geeft in één keer een volledige **gewogen permutatie** (= voorkeurlijst)

### Structuur van de voorkeurlijsten
- **Positie 1**: gesampled uit `vk_1` verdeling (werkelijke 1e-keuze populariteit)
- **Positie 2**: gesampled uit `vk_2` verdeling, conditioneel op 1e keuze ≠ 2e keuze
- **Positie 3**: gesampled uit `vk_3` verdeling, conditioneel op ≠ keuze 1 en 2
- **Positie 4-12**: gewogen random permutatie van resterende scholen op basis van totale populariteit

### Niet-overvraagde scholen
Belangrijk inzicht uit validatie: **30 van de 41 scholen** zijn niet overvraagd (vk_1 ≤ capaciteit). Leerlingen die zo'n school als 1e keuze hebben, worden **altijd direct geplaatst**, ongeacht lotnummer. We verwerken deze eerst, vóór de DA-STD loting.

### Capaciteit
We gebruiken `gpl_2025` (werkelijk geplaatst) als capaciteit i.p.v. `cap_2025`, omdat die al de ~4% plaatsingsgarantie-verhoging bevat.

### Validatie
- **Totale plaatsingen per school:** exact gelijk aan werkelijkheid (verschil = 0.0 voor alle 41 scholen) ✅
- **Rondes-verdeling (1e/2e/3e/4e+):** wijkt af van werkelijkheid omdat we de correlatie tussen individuele voorkeurslijsten niet kennen — we weten niet welke specifieke combinaties leerlingen maken. Dit beïnvloedt de simulatie van jouw uitkomst minimaal.

## Data

De data komt uit `vwo_scholen.csv` — tabel 17 uit het L&M 2025 verslag. Per school hebben we:
- `cap_2025` / `gpl_2025` — capaciteit en werkelijk geplaatst (gpl bevat de ~4% verhoging)
- `vk_1/2/3` — hoeveel leerlingen de school als 1e/2e/3e keuze hadden
- `nvk_1/2/3/4plus` — van de geplaatsten, via welke voorkeurspositie

## Beperkingen

**Wat het model goed doet:**
- De totale plaatsingsaantallen per school kloppen exact met de werkelijkheid
- De kansen nemen netjes af naarmate het lotnummer hoger wordt
- Populaire maar kleine scholen (zoals St. Nicolaaslyceum) worden correct vroeg vol
- De cluster-indeling zorgt dat leerlingen realistischer gedrag vertonen op posities 4-12

**Waar het model minder nauwkeurig is:**

1. **We weten niet wie welke combinatie kiest.** We weten wel dat 211 leerlingen St. Nicolaaslyceum als eerste keuze hadden en 254 leerlingen Barlaeus — maar niet of dat dezelfde leerlingen zijn. In werkelijkheid zijn er groepen leerlingen met vergelijkbare smaken.
2. **De cluster-indeling is een aanname.** We hebben vijf typen onderscheiden (gymnasium, traditioneel lyceum, montessori, vernieuwend, rest), maar in werkelijkheid zijn de overeenkomsten subtieler. De factor van drie keer hogere kans voor hetzelfde cluster is een schatting, geen gemeten waarde.
3. **Keuzes 4 tot 12 zijn onzeker.** We hebben hiervoor geen data. De cluster-aanpak is beter dan willekeurig, maar nog steeds een vereenvoudiging van de werkelijkheid.
4. **We gebruiken cijfers uit 2025 voor 2026.** De capaciteit, populariteit en het aantal leerlingen kan volgend jaar anders zijn.
5. **Bijzondere regels tellen niet mee.** Leerlingen met voorrang (via de Kopklas of speciaal onderwijs) en handmatige plaatsingen zijn niet meegenomen. Die nemen maximaal 2% van de plekken in.

**Conclusie:** de simulatie geeft een **goede indicatie** van je kansen, maar is geen exacte voorspelling. Gebruik het als hulpmiddel bij het nadenken over je lijstvolgorde, niet als garantie.
