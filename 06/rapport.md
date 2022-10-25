---
title: IDATT2103, øving 6
author: Jakob Grønhaug (jakobkg@stud.ntnu.no)
---

# Vikartjeneste

## a) UML

![Modellen i UML](modell.pdf)

## b) ER

Kandidat(\
\ \hspace{1cm}\underline{id}: int, \
\ \hspace{1cm}fornavn: string, \
\ \hspace{1cm}etternavn: string, \
\ \hspace{1cm}telefon: string, \
\ \hspace{1cm}epost: string \
)

Bedrift(\
\ \hspace{1cm}\underline{org\_nr}: string \
\ \hspace{1cm}navn: string \
\ \hspace{1cm}telefon: string \
\ \hspace{1cm}epost: string \
)

Oppdrag(\
\ \hspace{1cm}\underline{id}: int, \
\ \hspace{1cm}kvalifikasjon\*, \
\ \hspace{1cm}start: date, \
\ \hspace{1cm}slutt: date, \
\ \hspace{1cm}ansatt\*: int \
)

Kvalifikasjon( \
\ \hspace{1cm}\underline{id}: int,
\ \hspace{1cm}\underline{navn}
)

Kandidat_Kvalifikasjon( \
\ \hspace{1cm}\underline{kandidat\*}: int, \
\ \hspace{1cm}\underline{kvalifikasjon\*}: int \
)

Her gir det mening å la noen fremmednøkler være `NULL`, for eksempel vil et oppdrags ansatte vikar være `NULL` frem til noen faktisk blir ansatt, og om oppdraget ikke krever noen kvalifikasjoner kan også dette feltet være `NULL`
\newpage{}
## c) CREATE TABLE

```sql
create table kvalifikasjon(
    id int unsigned auto_increment primary key,
    navn tinytext
);

create table kandidat(
    id int unsigned auto_increment primary key,
    fornavn tinytext,
    etternavn tinytext,
    telefon tinytext,
    epost tinytext
);

create table bedrift(
    orgnr varchar(9) primary key,
    navn tinytext,
    telefon tinytext,
    epost tinytext
);

create table kandidat_kvalifikasjon(
    kval_id int unsigned,
    kand_id int unsigned,
    primary key (kval_id, kand_id),
    foreign key (kval_id)
        references kvalifikasjon(id),
    foreign key (kand_id)
        references kandidat(id)
);

create table oppdrag(
    id int unsigned auto_increment primary key,
    bedrift varchar(9),
    kvalifikasjon int unsigned,
    oppstart date,
    slutt date,
    ansatt int unsigned,
    foreign key (bedrift)
        references bedrift(orgnr),
    foreign key (kvalifikasjon)
        references kvalifikasjon(id),
    foreign key (ansatt)
        references kandidat(id)
);
```
\newpage{}
## d) SELECT

1. 
    ```sql
    select 
        navn, telefon, epost
    from
        bedrift;
   ```

2. 
    ```sql
    select
        oppdrag.id as oppdrag,
        bedrift.navn as bedrift,
        bedrift.telefon as telefon
    from
        oppdrag natural join bedrift;
   ```
3. 
    ```sql
    select
        kandidat.fornavn as navn,
        kandidat.id as kanidatid,
        kvalifikasjon.navn as kvalifikasjon,
        kvalifikasjon.id as kvalid
    from
        kandidat
    natural join kandidat_kvalifikasjon
    natural join kvalifikasjon;
   ```

4. 
    ```sql
    select
        kandidat.fornavn as navn,
        kandidat.id as kandidatid,
        kvalifikasjon.navn as kvalifikasjon,
        kvalifikasjon.id as kvalid
    from
        kandidat
    left join kandidat_kvalifikasjon
    natural join kvalifikasjon;
   ```

5. 
    ```sql
    select
        fornavn,
        etternavn,
        bedrift.navn as bedrift,
        oppdrag.slutt as sluttdato
    from
        oppdrag
    where
        ansatt = ANGITT_ID
        and
        slutt not null
    natural join kandidat
    natural join bedrift;
   ```
