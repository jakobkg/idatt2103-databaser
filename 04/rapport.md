---
title: IDATT2103, øving 4
author: Jakob Grønhaug (jakobkg@stud.ntnu.no)
---

# 1. SQL inkl. VIEW

a. Alt om ordre for leverandør 44

```sql
select
    *
from
    ordrehode natural join ordredetalj
where
    ordrehode.levnr = 44;
```

```
+---------+------------+-------+--------+-------+---------+
| ordrenr | dato       | levnr | status | delnr | kvantum |
+---------+------------+-------+--------+-------+---------+
|      13 | 1986-09-13 |    44 | p      | 51173 |      20 |
|      14 | 1986-12-17 |    44 | p      |   201 |     100 |
|      14 | 1986-12-17 |    44 | p      |   202 |     100 |
|      14 | 1986-12-17 |    44 | p      | 51173 |      30 |
|      15 | 1987-01-03 |    44 | p      |   201 |     100 |
|      15 | 1987-01-03 |    44 | p      |   202 |     100 |
+---------+------------+-------+--------+-------+---------+
```

b. Leverandører som kan levere del 1

```sql
select
    navn as Navn,
    levby as "By"
from
    prisinfo natural join levinfo
where
    delnr = 1;
```
\newpage
```
+---------------------+------+
| Navn                | By   |
+---------------------+------+
| Kontorekspressen AS | Oslo |
| Kontorutstyr AS     | Ås   |
+---------------------+------+
```

c. Billigste leverandør av del 201

```sql
select
    levnr, navn, pris
from
    levinfo natural join prisinfo
where
    pris =
        (select 
            min(pris)
        from
            prisinfo
        where
            delnr = 201)
```

```
+-------+------------------+------+
| levnr | navn             | pris |
+-------+------------------+------+
|    44 | Billig og Bra AS |  1.6 |
+-------+------------------+------+
```

d. Ordre 16

```sql
select
    ordrenr as Ordre,
    dato as Dato,
    delnr as Del,
    beskrivelse as Beskrivelse,
    kvantum as Antall,
    pris as Enhetspris,
    round(kvantum * pris, 2) as Totalpris
from
    ordrehode
    natural join ordredetalj
    natural join prisinfo
    natural join delinfo
where
    ordrenr = 16;
```

```
+-------+------------+-------+-------------------+--------+------------+-----------+
| Ordre | Dato       | Del   | Beskrivelse       | Antall | Enhetspris | Totalpris |
+-------+------------+-------+-------------------+--------+------------+-----------+
|    16 | 1987-01-31 |   201 | Svarte kulepenner |     50 |        1.9 |        95 |
|    16 | 1987-01-31 |   202 | Blå kulepenner    |     50 |        6.5 |       325 |
|    16 | 1987-01-31 |  1909 | Skriveunderlag    |     10 |       0.85 |       8.5 |
|    16 | 1987-01-31 | 51173 | Binders           |     20 |       0.57 |      11.4 |
+-------+------------+-------+-------------------+--------+------------+-----------+
```

e. Deler dyrere enn delen med katalognummer X7770

```sql
select
    delnr, levnr
from
    prisinfo
where
    pris >
        (select
            pris
        from
            prisinfo
        where
            katalognr = "X7770");
```

```
+-------+-------+--------+
| delnr | levnr | pris   |
+-------+-------+--------+
|     1 |     6 |    120 |
|     1 |     9 | 219.99 |
|     3 |     6 |   1199 |
|     3 |     9 |   1050 |
|     3 |    82 |   1299 |
|     4 |     6 |    550 |
|     4 |    82 |    899 |
|   202 |     6 |    6.5 |
| 51200 |     6 |   7.35 |
+-------+-------+--------+
```
\newpage
f.
i)
```sql
create table fylker(
    fylke varchar(255) not null,
    bynavn varchar(255) not null,
    primary key (bynavn)
);

create table levinfo_fylkefri(
    levnr int unsigned not null,
    navn tinytext not null,
    adresse tinytext not null,
    levby varchar(255) not null,
    postnr int unsigned not null,
    primary key (levnr),
    foreign key (levby) references fylker(bynavn)
);

insert into fylker values ("Oslo", "Oslo");
insert into fylker values ("Østfold", "Ås");
insert into fylker values ("Telemark", "Ål");
insert into fylker values ("S-Trøndelag", "Trondheim");

insert into levinfo_fylkefri values (
    6, "Kontorekspressen AS", "Skolegata 3", "Oslo", 1234 
);
insert into levinfo_fylkefri values (
    9, "Kontorutstyr AS", "Villa Villekulla", "Ås", 1456
);
insert into levinfo_fylkefri values (
    12, "Mister Office AS", "Storgt 56", "Ås", 1456
);

insert into levinfo_fylkefri values (
    44, "Billig og Bra AS", "Aveny 56", "Oslo", 1222
);

insert into levinfo_fylkefri values (
    81, "Kontorbutikken AS", "Gjennomveien 78", "Ål", 3345
);

insert into levinfo_fylkefri values (
    82, "Kontordata AS", "Åsveien 178", "Trondheim", 7023
);

```

```
+--------------+-----------+
| fylke        | bynavn    |
+--------------+-----------+
| Telemark     | Ål        |
| Østfold      | Ås        |
| Oslo         | Oslo      |
| S-Trøndelag  | Trondheim |
+--------------+-----------+
+-------+---------------------+------------------+-----------+--------+
| levnr | navn                | adresse          | levby     | postnr |
+-------+---------------------+------------------+-----------+--------+
|     6 | Kontorekspressen AS | Skolegata 3      | Oslo      |   1234 |
|     9 | Kontorutstyr AS     | Villa Villekulla | Ås        |   1456 |
|    12 | Mister Office AS    | Storgt 56        | Ås        |   1456 |
|    44 | Billig og Bra AS    | Aveny 56         | Oslo      |   1222 |
|    81 | Kontorbutikken AS   | Gjennomveien 78  | Ål        |   3345 |
|    82 | Kontordata AS       | Åsveien 178      | Trondheim |   7023 |
+-------+---------------------+------------------+-----------+--------+
```

ii)

```sql
create view levinfo_view as
select
    levnr, navn, adresse, levby, fylke, postnr
from
    levinfo_fylkefri left join fylker
    on
        levinfo_fylkefri.levby = fylker.bynavn;
```

```
+-------+---------------------+------------------+-----------+--------------+--------+
| levnr | navn                | adresse          | levby     | fylke        | postnr |
+-------+---------------------+------------------+-----------+--------------+--------+
|     6 | Kontorekspressen AS | Skolegata 3      | Oslo      | Oslo         |   1234 |
|     9 | Kontorutstyr AS     | Villa Villekulla | Ås        | Østfold      |   1456 |
|    12 | Mister Office AS    | Storgt 56        | Ås        | Østfold      |   1456 |
|    44 | Billig og Bra AS    | Aveny 56         | Oslo      | Oslo         |   1222 |
|    81 | Kontorbutikken AS   | Gjennomveien 78  | Ål        | Telemark     |   3345 |
|    82 | Kontordata AS       | Åsveien 178      | Trondheim | S-Trøndelag  |   7023 |
+-------+---------------------+------------------+-----------+--------------+--------+
```

Dette er et view basert på mer enn en tabell, derfor er viewet ikke oppdaterbart. Kun SELECT vil fungere mot dette viewet, ikke DELETE, INSERT eller UPDATE.
\newpage
g. Hvilke byer har ingen levernadører ved sletting av leverandører uten prisinfo

```sql
select
    levby
from
    levinfo
where
    levby not in (
        select
            levby
        from prisinfo
            natural join levinfo
        );
```

```
+-------+
| levby |
+-------+
| Ål    |
+-------+
```

h. Billigste leverandør av ordre 18
Lager først et view av alle som kan levere noen av delene i ordre 18

```sql
create view ordre18 as
    select prisinfo.levnr,
        ordredetalj.delnr,
        pris as stykkpris,
        kvantum as antall,
        pris*kvantum as totalpris
    from levinfo
        natural join ordredetalj
        natural join prisinfo
    where ordrenr = 18;
```

```
+-------+-------+-----------+--------+-----------+
| levnr | delnr | stykkpris | antall | totalpris |
+-------+-------+-----------+--------+-----------+
|     6 |     3 |      1199 |      2 |      2398 |
|     9 |     3 |      1050 |      2 |      2100 |
|    82 |     3 |      1299 |      2 |      2598 |
|     6 |     4 |       550 |      8 |      4400 |
|    82 |     4 |       899 |      8 |      7192 |
+-------+-------+-----------+--------+-----------+
```

Lager så et view av alle leverandører som kan levere to av to deler til denne ordren

```sql
create view totallev_18 as
    select
        levnr, sum(totalpris) as total
    from
        ordre18
    group by
        levnr
    having
        count(levnr) = 2;
```

```
+-------+--------------+-------+
| levnr | count(levnr) | total |
+-------+--------------+-------+
|     6 |            2 |  6798 |
|    82 |            2 |  9790 |
+-------+--------------+-------+
```

Finner så til sist den leverandøren i denne tabellen som har lavest totalpris

```sql
select
    *
from
    totallev_18
where
    total = (
        select
            min(total)
        from
            totallev_18
        );
```

```
+-------+-------+
| levnr | total |
+-------+-------+
|     6 |  6798 |
+-------+-------+
```
\newpage
# 2. NULL

a. NULL i union

```sql
select
    *
from
    forlag
where
    telefon like "2%"
union
select
    *
from
    forlag
where
    telefon not like "2%";
```

Dette resultatet har ikke med forlaget der telefon er `NULL`. For å få med dette må en tredje spørring til i unionen:

```sql
select
    *
from
    forlag
where
    telefon is null;
```
\newpage
b. Gjennomsnittsalder av forfattere
```sql
create view forfatter_alder as
select
    fornavn, etternavn, fode_aar,
    dod_aar as til_aar,
    dod_aar - fode_aar as alder
from
    forfatter
having
    fode_aar is not null
    and til_aar is not null
union
select
    fornavn, etternavn, fode_aar,
    2022 as til_aar,
    2022 - fode_aar as alder
from
    forfatter
where
    fode_aar > 1900
    and dod_aar is null;
```

```
+---------+-----------+----------+---------+-------+
| fornavn | etternavn | fode_aar | til_aar | alder |
+---------+-----------+----------+---------+-------+
| John    | Tool      |     1937 |    1969 |    32 |
| Knut    | Hamsund   |     1859 |    1952 |    93 |
| Jose    | Saramago  |     1942 |    2022 |    80 |
| Douglas | Coupland  |     1961 |    2022 |    61 |
| Henning | Mankell   |     1948 |    2022 |    74 |
+---------+-----------+----------+---------+-------+
```

Så kan man finne gjennomsnittet ved

```sql
select
    avg(alder) as snittalder
from
    forfatter_alder;
```

```
+------------+
| snittalder |
+------------+
|    68.0000 |
+------------+
```

c. Andel forfattere som ble med i forrige oppgave

```sql
create view medregnet as
select
    count(*) as antall
from
    forfatter_alder;
```

```sql
create view total as
select
    count(*) as antall
from
    forfatter;
```

```sql
select
    100 * medregnet.antall / total.antall as andel
from medregnet, total;
```

```
+---------+
| andel   |
+---------+
| 41.6667 |
+---------+
```

41.6667% av forfatterne ble med i forrige utregning.
