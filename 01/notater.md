---
author: Jakob Grønhaug (jakobkg@stud.ntnu.no)
title: IDATT2103, Øving 1
---

# Relasjoner med primærnøkler og fremmednøkler

borettslag($\underline{\text{navn}}$, postadresse, etableringsår)\
bygning($\underline{\text{adresse}}$, borettslag\*, etasjer)\
boenhet($\underline{\text{adresse*}}$, $\underline{\text{H-nummer}}$, areal, antall rom)\
andelseier($\underline{\text{medlemsnummer}}$, $\underline{\text{fornavn}}$, $\underline{\text{etternavn}}$, $\underline{\text{borettslag*}}$, boenhet\*)

Med disse relasjonene har man sammenheng en-til-mange mellom borettslag og bygninger, en-til-mange mellom borettslag og andelseier, og en-til-en uten eksistensavhengighet mellom boenhet og andelseier.

# SQL-setninger

## Opprette tabeller

### Borettslag

Borettslag antas å være unikt definert ved navn, en annen kandidatnøkkel er å opprette et eget ID-nummer per borettslag. I tillegg til navn har hvert borettslag en postadresse (obligatorisk) og et etableringsår (valgfritt).

```sql
create table borettslag (
    navn varchar(255) not null,
    postadresse tinytext not null,
    etablert int,
    primary key (navn));
```

### Bygning

En bygning er unikt identifisert på adressen sin, og har også et antall etasjer og en fremmednøkkel som kobler bygningen til borettslaget den er en del av. I tillegg har bygningen et antall etasjer som lagres her, da dette ikke nødvendigvis kan utledes fra noen annen data i relasjonene.


```sql
create table bygning (
    adresse varchar(255) not null,
    borettslag varchar(255) not null,
    etasjer int unsigned not null,
    primary key (adresse),
    foreign key (borettslag) references borettslag(navn));
```

## Boenhet

En boenhet er unikt definert gjennom kombinasjonen av adresse og H-nummer. For eksempel bor jeg i Elvegata 16, 7012 Trondheim, H0303 som er en unik identifikator for boenheten selv om det fins mange andre boenheter med H-nummer H0303, og mange andre boenheter med adresse Elvegata 16, 7012 Trondheim. En boenhet har også et antall rom og et areal, men disse er ikke påkrevd i tabellen for ågjøre det mulig å registrere en boenhet som er planlagt bygd men ikke har et kjent antall rom eller areal enda.

```sql
create table boenhet (
    adresse varchar(255) not null,
    h_nummer varchar(255) not null,
    areal int,
    rom int,
    primary key (adresse, h_nummer),
    foreign key (adresse) references bygning(adresse));
```

### Andelseier

Andelseiere er unikt definert gjennom en kombinasjon av borettslaget de er medlem av, et medlemsnummer og navnet deres. På denne måten kan to forskjellige personer ved navn Ola Nordmann være medlem i samme borettslag, og hvert borettslag kan ha et eget format på medlemsnumrene uten at det er noen fare for kollisjoner.

Ved bruk av `not null` er det sikret at hver andelseier *må* være del av et borettslag og ha et medlemsnummer, og *kan* men ikke *må* ha en boenhet.

```sql
create table andelseier (
    medlemsnummer int unsigned not null,
    fornavn tinytext not null,
    etternavn tinytext not null,
    borettslag varchar(255) not null,
    adresse varchar(255),
    h_nummer varchar(255),
    primary key (medlemsnummer, borettslag, fornavn, etternavn),
    foreign key (adresse, h_nummer) references boenhet(adresse, h_nummer),
    foreign key (borettslag) references borettslag(navn));
```

## Gyldig test-data

Her er noen `insert`-setninger som oppretter et borettslag med en bygning og flere bebodde boenheter, en ubebodd enhet og to medlemmer av laget som er potensielle kjøpere til den ubebodde boenheten.

### Borettslag

```sql
insert into borettslag values (
    "Sameiet Blokka 1",
    "Blokkvegen 1, 1234 Byen",
    1998
)
```

### Bygning

```sql
insert into bygning values (
    "Blokkvegen 1, 1234 Byen",
    "Sameiet Blokka 1",
    5
)
```

### Boenheter

```sql
insert into boenhet values (
    "Blokkvegen 1, 1234 Byen",
    "H0101",
    70,
    3
)
```

```sql
insert into boenhet values (
    "Blokkvegen 1, 1234 Byen",
    "H0102",
    50,
    2
)
```

```sql
insert into boenhet values (
    "Blokkvegen 1, 1234 Byen",
    "H0201",
    125,
    6
)
```

```sql
insert into boenhet values (
    "Blokkvegen 1, 1234 Byen",
    "H0302",
    63,
    3
)
```

### Andelseiere

```sql
insert into andelseier values (
    87249,
    "Boer",
    "Beboersen",
    "Sameiet Blokka 1",
    "Blokkvegen 1, 1234 Byen",
    "H0101"
)
```

```sql
insert into andelseier values (
    742,
    "Hyggelig",
    "Nabo",
    "Sameiet Blokka 1",
    "Blokkvegen 1, 1234 Byen",
    "H0102"
)
```

```sql
insert into andelseier values (
    4928,
    "Leie",
    "Hai",
    "Sameiet Blokka 1",
    "Blokkvegen 1, 1234 Byen",
    "H0201"
)
```

```sql
insert into andelseier values (
    423789,
    "Mulig",
    "Kjøper",
    "Sameiet Blokka 1",
    NULL,
    NULL
)
```

```sql
insert into andelseier values (
    923847,
    "Konkurrerende",
    "Boligsøker",
    "Sameiet Blokka 1",
    NULL,
    NULL
)
```

## Ugyldige insert-setninger

Bygning uten gyldig borettslag, feiler da dette borettslaget ikke eksisterer i borettslag-tabellen enda:

```sql
insert into bygning values (
    "Gateveien 123B, 1233 Nabobyen",
    "Veigata Velforening",
    1
)
```

Andelseier uten borettslag, feiler siden `andelseier(borettslag)` er bestemt `not null`:

```sql
insert into andelseier values (
    1,
    "Testdata",
    "Torgersen",
    NULL,
    NULL,
    NULL
)
```
