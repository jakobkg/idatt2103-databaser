---
title: IDATT2103, øving 2
author: Jakob Grønhaug
---

# Oppg 1
## a)
Spørring med seleksjon og projeksjon:
```sql
select forfatter_id, fornavn from forfatter where nasjonalitet="Britisk";
```
```
+--------------+---------+
| forfatter_id | fornavn |
+--------------+---------+
|            2 | Ken     |
|            3 | Stephen |
|            6 | Nick    |
|            9 | Helen   |
+--------------+---------+
```

## b)
Spørring med produkt:
```sql
select * from bok, forlag;
```

Resultatet er en tabell som inneholder alle mulige kombinasjoner av tupler fra de to tabellene. Antallet kolonner er summen av antallet kolonner i hver tabell, og antallet tupler er produktet av antallet tupler i hver tabell.

## c)
### Likhetsforening
```sql
select 
    bok.tittel, bok.utgitt_aar, forlag.forlag_navn
from
    bok, forlag
where
    bok.forlag_id = forlag.forlag_id;
```

Resultater er en ny tabell der data fra tabellene for forlag og bøker er slått sammen slik at boktittel, utgivelsesår og forlag står samlet i en tabell i stedet for to separate tabeller.

### Naturlig forening
```sql
select
    bok.tittel, bok.utgitt_aar, forlag.forlag_navn
from 
    bok natural join forlag;
```

Denne spørringen gjør det samme som den forrige, men bruker i stedet `natural join` for å kombinere de to tabellene. Dette fungerer fordi felles-kolonnen heter `forlag_id` i begge tabeller.

## d)
To attributter er unionkompatible om de er av samme type og domene, og to relasjoner er unionkompatible om de har utelukkende unionkompatible attributter. Union og snitt krever at relasjonene er unionkompatible.

I databasen gitt skriptet i oppgaven er for eksempel alle navne-feltene i forfatter og konsulent unionkompatible.

## Union
```sql
select fornavn from konsulent
union
select etternavn from forfatter;
```
Gir en ny tabell med alle fornavnene til konsulenter og alle etternavnene til forfattere, for å samle alle fornavn på ett sted.

## Snitt
```sql
select distinct
    konsulent.fornavn
from
    konsulent join forfatter
on
    konsulent.fornavn = forfatter.fornavn;
```
Gir alle fornavn som opptrer både blant konsulenter og forfattere (ingen overlapp i dette tilfellet).

# Oppg 2
## a)
For å finne navnene til forlagene kan man bruke følgende spørring:
```sql
select distinct forlag_navn from forlag;
```

## b)
```sql
select
    forlag_navn
from
    bok right join forlag
    on
        forlag.forlag_id = bok.forlag_id
where
    bok.forlag_id is null;
```

For å finne alle forlag som ikke har utgitt noen bøker brukes differanse (alle forlags `forlag_id` minus alle bøkers `forlag_id`)

## c)
```sql
select
    *
from
    forfatter
where
    fode_aar = 1948;
```
Her brukes en seleksjon (`where fode_aar = 1948`).

## d)
```sql
select
    forlag_navn, adresse
from
    forlag natural join bok
where
    bok.tittel = "Generation X";
```
I denne spørringen brukes en forening, en seleksjon og en projeksjon.

## e)
(MERK: Hamsun er registrert som Hamsund i skriptet lagt ved oppgaven)
```sql
select
    tittel
from
    bok natural join bok_forfatter natural join forfatter
where
    etternavn = "Hamsund";
```
Her brukes to foreninger, en seleksjon og en projeksjon.

## f)
```sql
select
    tittel, utgitt_aar, forlag_navn, adresse, telefon
from
    bok right outer join forlag
        on
            bok.forlag_id = forlag.forlag_id;
```
I denne spørringen brukes en projeksjon og en ytterforening.
