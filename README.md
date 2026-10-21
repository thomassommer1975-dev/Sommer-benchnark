# Sommer AI Benchmark 🥷

Et enkelt, versionsstyret og offentligt testbibliotek til uafhængig sammenligning af AI-modeller. Kun fiktive opgaver og offentlige testinstruktioner hører hjemme her.

## Sådan bruges laboratoriet

1. Åbn [ACTIVE.md](ACTIVE.md) og find den aktive, **låste** benchmarkversion.
2. Giv hver kandidat samme låste opgave i en helt ny, kontekstfri samtale. Ingen web, værktøjer eller repo-adgang ud over at hente den offentlige opgavetekst, medmindre netop den version udtrykkeligt kræver det.
3. Kandidaten svarer kun i sin egen chat, ikke i GitHub. Resultatet afleveres anonymt som C1, C2 osv. til dommeren.
4. Modelidentiteter afsløres først, når alle vurderinger er låst.

## Regler

- Opgaver i `benchmarks/` er frosne, når de frigives. Ret aldrig en gammel udgave; opret en ny version.
- **Ingen facit, skjulte tests, private besvarelser, modelnøgler, adgangstokens eller persondata i dette offentlige repository.**
- Benchmarkdesigneren deltager ikke i samme runde.
- Benchmarks adskilles fra alle virkelige Sommer-projekter og produktionsmiljøer.
- Korrekthed og efterprøvbarhed vægter højere end velformuleret tekst. Dokumentér hvilke tests der faktisk er kørt; opfind ingen resultater.

Se [ACTIVE.md](ACTIVE.md) for status og [prompts/LAUNCHER.md](prompts/LAUNCHER.md) for den genbrugelige kandidatprompt.

