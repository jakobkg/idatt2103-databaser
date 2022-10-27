---
title: IDATT2103 - Øving 5
author: Jakob Grønhaug (jakobkg@stud.ntnu.no)
classoption: table
header-includes: |
    \rowcolors{2}{gray!10}{gray!15}
---

# Normalisering

## Kandidatnøkler

Forslag til kandidatnøkler:

- `utleie_id`, en kombinasjon av eiendom_id, kunde_id, fra_uke og til_uke
- `kunde_id`, representerer en kunde, er entydig for denne kunden
- `eier_id`, representerer eieren av boligen som leies ut, er entydig for denne eieren
- `eiendoms_id`, representerer eiendommen som leies ut, er entydig for denne eiendommen

## Problemer med den gitte relasjonen

Den oppgitte relasjonen i oppgaveteksten kommer til å ha problemer med registrering og sletting pga stor grad av redundans, da det ikke er noe enkelt felt som unikt identifiserer en utleie-avtale. Å legge til data blir unødvendig vanskelig da man må sørge for at alle kundedata, eierdata og eiendom-data føres inn *nøyaktig* likt hver gang for å kunne finne dem igjen ved en senere anledning. På samme måte vil det å finne riktig booking for sletting være komplisert da man er nødt til å slå opp på både kunde, eiendom, eier og tidsperiode for å kunne unikt identifisere en leieavtale.

\newpage{}

## Funksjonelle avhengigheter

## Oppsplitting

\begin{minipage}[c]{0.25\textwidth}
	\centering
	\begin{tabular}{|l|}
		\hline
		\bf{Kunde} \\
		\hline
		kunde\_id \\
		navn \\
		telefonnummer \\
		hjemadresse \\
		\hline
	\end{tabular}
\end{minipage}
\begin{minipage}[c]{0.25\textwidth}
	\centering
	\begin{tabular}{|l|}
		\hline
		\bf{Eier} \\
		\hline
		eier\_id \\
		navn \\
		telefonnummer \\
		hjemadresse \\
		\hline
	\end{tabular}
\end{minipage}
\begin{minipage}[c]{0.25\textwidth}
	\centering
	\begin{tabular}{|l|}
		\hline
		\bf{Eiendom} \\
		\hline
		eiendom\_id \\
		adresse \\
		eier\_id \\
		pris \\
		\hline
	\end{tabular}
\end{minipage}
\begin{minipage}[c]{0.25\textwidth}
	\centering
	\begin{tabular}{|l|}
		\hline
		\bf{Leieavtale} \\
		\hline
		eiendom\_id \\
		kunde\_id \\
		fra\_dato \\
		til\_dato \\
		\hline
	\end{tabular}
\end{minipage}

Jeg velger å dele tabellen i fire. Leietakere, utleiere, eiendommer og leieavtaler får hver sin tabell, med fremmednøkler der det trengs for å kunne gjenskape den originale relasjonen om ønskelig.

## Relasjoner på BCNF

```
KUNDE(kunde_id, kunde_adresse, kunde_tlf, kunde_navn)

EIER(eier_id, eier_adresse, eier_tlf, eiendom_id, eier_navn)

EIENDOM(eiendom_id, eiendom_adresse)

LEIEAVTALE(eiendom_id, fra_uke, til_uke, kunde_id, pris)
```

## 1NF-2NF-3NF

### 1NF

Krever kun atomiske verdier og en primærnøkkel per relasjon

```
KUNDE(kunde_id, kunde_navn, kunde_adresse, kunde_tlf)

EIER(eier_id, eier_navn, eier_adresse, eier_tlf, eiendom_id)

EIENDOM(eiendom_id, eiendom_adresse, fra_uke, til_uke, pris)
```

### 2NF

Ingen sammensatte primærnøkler, oppnøs ved å dele opp Eiendom-tabellen

```
EIENDOM(eiendom_id, eiendom_adresse, pris)

LEIEAVTALE(eiendom_id, fra_uke, til_uke, kunde_id)
```

### 3NF

En tabell er på 3NF hvis 2NF er oppfylt og det ikke finnes noen transitive determineringer mellom attributter som ikke er kandidatnøkler. I oppsplittingene som er gjort finnes det ingen transitive determineringer, og vi har dermed oppnådd 3NF ved:

```
KUNDE(kunde_id, kunde_navn, kunde_adresse, kunde_tlf)

EIER(eier_id, eier_navn, eier_adresse, eier_tlf, eiendom_id)

EIENDOM(eiendom_id, eiendom_adresse)

UTLEIEAVTALER(eiendom_id, fra_uke, til_uke, kunde_id, pris)
```

# Transaksjoner

## Teori
1. Hvilke typer låser har databasesystemene?
	Skrivelås og leselås

2. Hva er grunnen til at at man gjerne ønsker lavere isolasjonsnivå enn SERIALIZABLE?
	Fordi SERIALIZABLE er et for strengt krav som hindrer nær alle former for paralell utføring av spørringer og begrenser ytelse betydelig.

3. Hva skjer om to pågående transaksjoner med isolasjonsnivå serializable prøver select sum(saldo) from konto?
	Begge spørringer gjennomføres, siden dette kun er lesing fra tabellen.

4. Hva er to-fase-låsing?
	To-fase-låsing er en strategi der man setter og fjerner låser i to separate "faser" av en transaksjon.

5. Hvilke typer samtidighetsproblemer (de har egne navn) kan man få ved ulike isolasjonsnivåer? Hva er optimistisk låsing/utførelse? Hva kan grunnen til å bruke dette være?
	
	- Dirty read
		- Transaksjon B kan se data som transaksjon A har endret, men ikke bekreftet (commited). Dvs at dataene kan bli rullet tilbake.
	- Non-repeatable read
		- B leser en verdi som A senere endrer. Hvis B leser verdien på nytt i samme transaksjon, men nå ser den endringen A foretok, har vi et eksempel på ikke-repeterende lesning.
	- Phantom read
		- B leser et sett med rader begrenset av where-uttrykk.
		- A legger inn data som passer inn i intervallet gitt av where-uttrykket.
		- B leser på nytt, men ser nå også de dataene A la inn (data som tidligere ikke fantes, fantomdata).
	
	Optimistisk låsing/utførelse er når man forsøker å sette en lås men det ikke kan gjennomføres, og man prøver på nytt rett etterpå uten å endre på transaksjonen. Dette kan fungere dersom grunnen til at transaksjonen ikke gikk gjennom var på grunn av en annen transaksjon som foregikk samtidig, heller enn en feil med den forsøkte transaksjonen. 

6. Hvorfor kan det være dumt med lange transaksjoner (som tar lang tid)? Vil det være lurt å ha en transaksjon hvor det kreves input fra bruker? \
	Lange transaksjoner kan føre til at transaksjoner fra andre brukere ikke kommer gjennom, som kan forstyrre arbeidsflyt. En annen grunn til at lange transaksjoner bør unngås er at dersom det skjer en feil under kjøringen, så vil alt arbeidet forsvinne. Derfor bør  man dele opp transaksjonene i mindre deler der det er fornuftig. Det vil mest sannsynlig ikke være nyttig å ha en transaksjon som krever input fra en bruker, fordi dette kommer mest sannsynlig til å øke tidsbruket til transaksjonen og føre til unødvendig lang tid der ressurser er låst for andre typer transaksjoner.


## Oppgaver
1. I steg 5 blir klient 1 hengende frem til klient 2 sin transaksjon var gjennomført. Det er fordi klient 2 har satt en delt lås (leselås) i steg 3. Så lenge klient 2 bruker repeatable read eller et “mindre strengt” isolasjonsnivå, vil klient 1 kunne utføre steg 5 uten å måtte vente til transaksjonen til klient 2 er ferdig.

2.
	a) Klient 2 får ikke utført steg 4 før klient 1 har utført steg 6. Grunnen til at det må være sånn er at man ikke skal kunne overskrive transaksjonene til en annen klient da dette kan skape problemer ved rollback.
	b) Vi får en deadlock, som vil si at klient 1 venter på klient 2, og klient 2 venter på klient 1. Da vil ingen av transaksjonene kunne gjennomføres, så serveren tvinger rollback av den ene transaksjonen. I oppgave a) så måtte kun en klient vente på den andre, mens i denne oppgaven måtte begge vente på hverandre, som ga deadlock. Det vil ikke hjelpe å endre isolasjonsnivå på klientene, alle isolasjonsnivå vil ende i deadlock.
 
3.
	Ved steg 3 får klient 1 269 som sum av saldoene, og i steg 5 og 7 får man 279, selv om klient 2 ikke har committed i steg 5. Dette er på grunn av at klient 1 har read uncommitted som isolasjonsnivå, og kan dermed lese endringer som klient 2 har gjort uten at de er committed. Om klient 1 bruker et annet isolasjonsnivå vil denne enten måtte vente på at klient 2 committer i steg 5, eller så vil klient 1 få 269 som resultat i steg 5.


4.  \

+---------------------------------------+-----------------------------------+
|**Klient 1**							|**Klient 2**						|
+=======================================+===================================+
|Sett isolasjonsnivå repeatable read	|Sett isolasjonsnivå serializable	|
+---------------------------------------+-----------------------------------+
|Start transaksjon						|Start transaksjon					|
+---------------------------------------+-----------------------------------+
|`select * from konto where kontonr>3;`	| 									|
+---------------------------------------+-----------------------------------+
| 										|`into konto add values(null, 199);`|
+---------------------------------------+-----------------------------------+
|`select * from konto where kontonr>3`;	| 									|
+---------------------------------------+-----------------------------------+
| 										|`commit;`							|
+---------------------------------------+-----------------------------------+
|`select * from konto where kontonr>3;`	| 									|
+---------------------------------------+-----------------------------------+
|`commit;` 								|									|
+---------------------------------------+-----------------------------------+
	

