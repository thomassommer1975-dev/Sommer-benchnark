# B001-v1 — Daglige sensormålinger (DRAFT)

**Status: KUN DRAFT.** Denne opgave er ikke frigivet. Den er et selvstændigt,
fiktivt benchmark til sammenligning af Claude, Gemini, GPT og Codex. Benchmarkens
designer deltager ikke. GitHub Copilot har alene udformet opgaven.

## Kandidatopgave

Et fiktivt netværk af fælleshaver bruger små sensorer til at måle jordfugtighed.
Sensorer sender JSON-hændelser til en lokal rapportgenerator. En fil kan indeholde
hændelser fra flere sensorer, være usorteret og indeholde gentagne hændelser,
fordi en sensor kan gensende data efter en afbrudt forbindelse.

Nedenfor er hele den relevante Python 3-implementation. Den bruger kun
standardbiblioteket. Du skal analysere og forbedre den efter specifikationen
nedenfor. Du behøver ikke internet, værktøjer eller andre filer.

```python
from __future__ import annotations

import json
from dataclasses import dataclass
from datetime import date, datetime, timezone
from typing import Iterable


@dataclass(frozen=True)
class Reading:
    device_id: str
    event_id: str
    timestamp: datetime
    moisture: int


@dataclass(frozen=True)
class DailySummary:
    device_id: str
    day: date
    count: int
    mean: float
    minimum: int
    maximum: int


def parse_record(value: object) -> Reading:
    if not isinstance(value, dict):
        raise ValueError("record must be an object")

    device_id = value["device_id"]
    event_id = value["event_id"]
    timestamp_text = value["timestamp"]
    moisture = value["moisture"]

    if not isinstance(device_id, str) or not device_id.strip():
        raise ValueError("invalid device_id")
    if not isinstance(event_id, str) or not event_id.strip():
        raise ValueError("invalid event_id")
    if not isinstance(timestamp_text, str):
        raise ValueError("invalid timestamp")

    try:
        timestamp = datetime.fromisoformat(
            timestamp_text.replace("Z", "+00:00")
        )
    except ValueError as exc:
        raise ValueError("invalid timestamp") from exc

    if timestamp.tzinfo is None:
        raise ValueError("timestamp must include an offset")
    if not isinstance(moisture, int) or not 0 <= moisture <= 100:
        raise ValueError("moisture must be an integer from 0 to 100")

    return Reading(device_id, event_id, timestamp, moisture)


def load_records(lines: Iterable[str]) -> tuple[list[Reading], list[tuple[int, str]]]:
    records: list[Reading] = []
    errors: list[tuple[int, str]] = []

    for line_number, line in enumerate(lines, start=1):
        if not line.strip():
            continue
        try:
            records.append(parse_record(json.loads(line)))
        except (json.JSONDecodeError, TypeError, ValueError) as exc:
            errors.append((line_number, str(exc)))

    return records, errors


def summarize(
    records: Iterable[Reading],
    since: datetime | None = None,
    until: datetime | None = None,
) -> list[DailySummary]:
    groups: dict[tuple[str, date], list[int]] = {}
    seen_events: set[str] = set()

    for reading in records:
        if since is not None and reading.timestamp < since:
            continue
        if until is not None and reading.timestamp >= until:
            continue

        if reading.event_id in seen_events:
            continue
        seen_events.add(reading.event_id)

        key = (reading.device_id, reading.timestamp.date())
        groups.setdefault(key, []).append(reading.moisture)

    result = []
    for (device_id, day), values in groups.items():
        result.append(
            DailySummary(
                device_id=device_id,
                day=day,
                count=len(values),
                mean=sum(values) // len(values),
                minimum=min(values),
                maximum=max(values),
            )
        )

    return sorted(result, key=lambda item: (item.device_id, item.day))
```

## Specifikation for ønsket adfærd

1. Hver ikke-tom linje er ét JSON-objekt med felterne `device_id`, `event_id`,
   `timestamp` og `moisture`. Ekstra felter er tilladt og ignoreres.
2. `device_id` og `event_id` skal være strenge med mindst ét tegn, der ikke er
   whitespace. Identifikatorernes oprindelige tekst bevares.
3. `timestamp` skal være en gyldig ISO-8601 dato/tid med UTC-offset, inklusive
   `Z` for UTC. Naive datoer/tider er ugyldige. Alle gyldige tidspunkter
   normaliseres til UTC, før filtrering og gruppering.
4. `moisture` skal være et heltal mellem 0 og 100 inklusive. JSON-booleaner er
   ikke gyldige målinger.
5. `load_records` behandler linjer uafhængigt: ugyldige linjer giver en post i
   `errors` med linjenummer og en forklaring, men forhindrer ikke behandling af
   efterfølgende linjer. Tomme/whitespace-linjer ignoreres, men tæller stadig
   med i de efterfølgende linjenumre. Manglende felter og forkert JSON-type må
   ikke afbryde indlæsningen.
6. Hændelses-id'er er unikke **inden for én sensor**, ikke på tværs af sensorer.
   Ved gentagne `(device_id, event_id)`-par vinder den første gyldige post i
   inputrækkefølgen; senere poster for samme par ignoreres. Reglen gælder også,
   når posterne ligger uden for et eventuelt rapporteringsinterval.
7. `since` er en valgfri inkluderende nedre grænse, og `until` er en valgfri
   eksklusiv øvre grænse. Grænser skal være timezone-aware. Hvis begge er
   angivet, skal `since < until`; ellers afvises argumenterne med `ValueError`.
   Sammenligninger foretages som tidspunkter, ikke som tekst.
8. Rapporten grupperer accepterede målinger pr. sensor og UTC-kalenderdag.
   Hver gruppe indeholder antal, aritmetisk gennemsnit (afrundet til 2
   decimaler med Pythons `round`), minimum og maksimum. Tomme grupper
   udelades. Resultatet sorteres efter `device_id` og derefter dato.

## Aflevering

Svar på dansk eller engelsk. Hold svaret på omtrent højst 1.500 ord, plus kode
efter behov. Brug denne struktur:

1. **Fund:** Beskriv konkrete fejl/risici med henvisning til relevante dele af
   koden. Mærk hvert udsagn som verificeret ud fra koden, antagelse eller
   foreslået test.
2. **Rettelse:** Vis en fokuseret patch eller erstatningsfunktioner, og forklar
   kort vigtige designvalg. Bevar standardbiblioteket og de offentlige
   funktioners formål.
3. **Tests:** Beskriv konkrete testcases og forventede resultater. Angiv
   tydeligt, om tests kun foreslås, eller faktisk er kørt; opfind ikke
   testresultater.
4. **Selvkritik og prioritering:** Nævn vigtigste resterende begrænsning eller
   tvetydighed, samt hvad du ville prioritere ved mere tid.

## Offentlig vurdering — 100 point

Hvert område bedømmes separat med 0–20 point:

For hver 0–5-del gælder samme skala: 0 = fraværende eller forkert; 1 = mest
forkert; 2 = delvist korrekt med væsentlige mangler; 3 = overvejende korrekt
med mindre mangler; 4 = korrekt og dækkende; 5 = korrekt, dækkende og
underbygget med præcis begrundelse eller relevante testforslag.

| Kompetenceområde | Kriterier |
| --- | --- |
| Arkitektur/systemforståelse | 0–5: følger specifikationens dataflow; 0–5: skelner korrekt mellem indlæsning, validering, deduplikering, filtrering og aggregering; 0–5: patchen er afgrænset og bevarer nyttige grænseflader; 0–5: identificerer relevante antagelser og deres konsekvenser. |
| Kode/korrekthed | 0–5: robust håndtering af manglende/forkerte felter og JSON-typer; 0–5: korrekt validering af identifikatorer, tidsstempler og målinger; 0–5: korrekt deduplikering og intervalgrænser; 0–5: korrekt UTC-dag, statistik og sortering. |
| Fejlfinding/sikkerhed | 0–5: peger på konkrete fejl med kodebaseret begrundelse; 0–5: vurderer fejlhåndtering uden at skjule fejl i gyldige rækker; 0–5: vurderer relevante input-/ressourcerisici proportionelt; 0–5: undgår udokumenterede påstande og skelner fund fra antagelser. |
| Test/grænsetilfælde | 0–5: dækker offset/UTC-dag og intervalgrænser; 0–5: dækker identitets-scoping og gentagelser i forskellige rækkefølger; 0–5: dækker ugyldige felter, booleanmålinger og fortsættelse efter fejl; 0–5: dækker tomt input, statistik, rækkefølge og testenes forventninger. |
| Kritisk ræsonnement/prioritering | 0–5: prioriterer efter faktisk korrekthedsrisiko; 0–5: forklarer afvejninger uden unødig kompleksitet; 0–5: selvkritikken nævner en reel begrænsning; 0–5: foreslår en konkret, realistisk næste forbedring. |

Point gives for korrekt, specifik og efterprøvbar begrundelse, ikke for
ordmængde. En kandidat får ikke point for at hævde, at tests er kørt, uden
faktisk at have kørt dem.
