---
title: IDATT2103, Øving 3
author: Jakob Grønhaug (jakobkg@stud.ntnu.no)
---

## 1. Borettslag etablert 1975-1985
```sql
select
    *
from
    borettslag
where
    etabl_aar between 1975 and 1985;
```

```
+------------+--------------+-----------+--------+
| bolag_navn | bolag_adr    | etabl_aar | postnr |
+------------+--------------+-----------+--------+
| Tertitten  | Åsveien 100  |      1980 |   7020 |
+------------+--------------+-----------+--------+
```

## 2. Liste andelseiere

```sql
select
    fornavn || ' ' || etternavn ||
    ', ansiennitet: ' || ansiennitet || ' år' as Andelseier
from
    andelseier
order by
    ansiennitet desc;
```

```
+------------------------------------+
| Andelseier                         |
+------------------------------------+
| Anna Olsen, ansiennitet: 10 år     |
| Ingrid Olsen, ansiennitet: 8 år    |
| Arne Torp, ansiennitet: 7 år       |
| Arne Martinsen, ansiennitet: 4 år  |
| Even Trulsbo, ansiennitet: 3 år    |
+------------------------------------+
```

\newpage

## 3. Eldste borettslags etableringsår

```sql
select
    min(etabl_aar) as "Tidligste etablering"
from
    borettslag;
```

```
+----------------------+
| Tidligste etablering |
+----------------------+
|                 1980 |
+----------------------+
```

## 4. Adresser med boenhet med minst tre rom

```sql
select distinct
    bygn_adr as Adresse
from
    bygning
    natural join leilighet
where
    leilighet.ant_rom >= 3;
```

```
+---------------+
| Adresse       |
+---------------+
| Åsveien 100a  |
+---------------+
```

## 5. Antall bygninger i Tertitten

```sql
select
    count(*) as Antall
from
    bygning
where
    bolag_navn = "Tertitten";
```

```
+--------+
| Antall |
+--------+
|      4 |
+--------+
```

## 6. Antall bygninger i hvert borettslag

```sql
select
    borettslag.bolag_navn as Borettslag,
    count(bygning.bolag_navn) as "Antall bygg"
from
    borettslag
    left join bygning
    on
        borettslag.bolag_navn = bygning.bolag_navn
group by
    borettslag.bolag_navn;
```

```
+------------+-------------+
| Borettslag | Antall bygg |
+------------+-------------+
| Lerken     |           0 |
| Sisiken    |           1 |
| Tertitten  |           4 |
+------------+-------------+
```

## 7. Antall leiligheter i Tertitten

```sql
select
    count(*) as Antall
from
    leilighet
    natural join bygning
where
    bygning.bolag_navn = "Tertitten";
```

```
+--------+
| Antall |
+--------+
|      4 |
+--------+
```

\newpage

## 8. Høyeste etasje i Tertitten

```sql
select
    max(ant_etasjer) as "Høyeste etasje"
from
    bygning
where
    bolag_navn = "Tertitten";
```

```
+-----------------+
| Høyeste etasje  |
+-----------------+
|               6 |
+-----------------+
```

## 9. Andelseiere uten leilighet

```sql
select
    fornavn,
    etternavn,
    telefon
from
    andelseier
where
    and_eier_nr not in
        (select
            and_eier_nr
        from
            leilighet);
```

```
+---------+-----------+----------+
| fornavn | etternavn | telefon  |
+---------+-----------+----------+
| Arne    | Martinsen | 78555388 |
+---------+-----------+----------+
```

\newpage

## 10. Andelseiere per borettslag

```sql
select
    borettslag.bolag_navn as Borettslag,
    count(andelseier.and_eier_nr) as Deleiere
from
    borettslag
    left join andelseier
    on
        borettslag.bolag_navn = andelseier.bolag_navn
group by
    borettslag.bolag_navn
order by
    Deleiere;
```

```
+------------+----------+
| Borettslag | Deleiere |
+------------+----------+
| Lerken     |        0 |
| Sisiken    |        1 |
| Tertitten  |        4 |
+------------+----------+
```

## 11. Alle andelseiere, med leilighetsnummer

```sql
select
    fornavn || " " || etternavn as Navn, leil_nr
from
    andelseier
    left join leilighet
    on
        andelseier.and_eier_nr = leilighet.and_eier_nr;
```

```
+----------------+---------+
| Navn           | leil_nr |
+----------------+---------+
| Even Trulsbo   |       1 |
| Anna Olsen     |       2 |
| Ingrid Olsen   |       3 |
| Arne Torp      |       4 |
| Arne Martinsen |    NULL |
+----------------+---------+
```

## 12. Borettslag med leiligheter med 4 rom

```sql
select distinct
    bolag_navn
from
    leilighet
    natural join bygning
where
    ant_rom = 4;
```

```
Empty set
```

## 13. Andelseiere per postnummer

```sql
select
    poststed.postnr || " " || poststed as Sted,
    count(and_eier_nr) as "Antall eiere"
from
    poststed
    left join leilighet
    natural join bygning
    natural join andelseier
    on
        poststed.postnr = bygning.postnr
group by
    Sted;
```

```
+--------------------+--------------+
| Sted               | Antall eiere |
+--------------------+--------------+
| 2020 Skedsmokorset |            0 |
| 6408 Aureosen      |            0 |
| 7020 Trondheim     |            4 |
| 7033 Trondheim     |            0 |
+--------------------+--------------+
```
