---
title: IDATT2103 - Øving 7
author: 
- Jakob Grønhaug (jakobkg@stud.ntnu.no)
- Camilla Birkelund (camillkb@stud.ntnu.no)
header-includes:
    - \usepackage{multicol}
---

# Oppgave 1

## UML 

![Skikkelig kul modell](modell.pdf)

# Relasjonsmodell

\begin{minipage}[t]{.3\textwidth}
Kunde( \\
\underline{kundenr}: int \\
navn: string \\
adresse: string \\
)
\end{minipage}
\vrule
\hspace{.03\textwidth}
\begin{minipage}[t]{.3\textwidth}
Ordre( \newline
\underline{ordrenr}: int \newline
kundenr*: int \newline
ordredato: dato \newline
antatt\_levering: dato \newline
betalingsstatus: string \newline
rabattprosent: float \newline
)
\end{minipage}
\vrule
\hspace{.03\textwidth}
\begin{minipage}[t]{.3\textwidth}
Standardstol(\\
\underline{serienr}: int\\
modellnavn: string\\
type*: int\\
pris: int\\
lagerstatus: int\\
)
\end{minipage}
\hrule
\begin{minipage}[t]{.3\textwidth}
Stoltype(\\
\underline{id}: int\\
navn: string\\
)
\end{minipage}
\vrule
\hspace{.03\textwidth}
\begin{minipage}[t]{.3\textwidth}
Spesialstol(\\
\underline{serienr}: int\\
modellnavn: string\\
type*: int\\
)
\end{minipage}
\vrule
\hspace{.03\textwidth}
\begin{minipage}[t]{.3\textwidth}
Spesialstol\_deler(\\
\underline{stol\_serienr*}: int\\
\underline{del\_id*}: int\\
antall: int\\
)
\end{minipage}
\hrule
\begin{minipage}[t]{.3\textwidth}
Ordre\_Standardstol(\\
\underline{ordrenr*}: int\\
\underline{serienr*}: int\\
antall: int\\
)
\end{minipage}
\vrule
\hspace{.03\textwidth}
\begin{minipage}[t]{.3\textwidth}
Ordre\_Spesialstol(\\
\underline{ordrenr*}: int \\
\underline{serienr*}: int\\
antall: int\\
)
\end{minipage}
\vrule
\hspace{.03\textwidth}
\begin{minipage}[t]{.3\textwidth}
Arbeidsstasjon(\\
\underline{stasjonsnr}: int\\
plassering: string\\
)
\end{minipage}
\hrule
\begin{minipage}[t]{.3\textwidth}
Stoldel(\\
\underline{serienr}: int\\
navn: string\\
farge: string\\
pris: float\\
beskrivelse: string\\
stofftype\*: int\\
stoffmengde: float\\
stasjon\*: int\\
lagerstatus: int\\
)
\end{minipage}
\vrule
\hspace{.03\textwidth}
\begin{minipage}[t]{.3\textwidth}
Stofftype(\\
\underline{serienr}: int\\
navn: string\\
farge: string\\
meterpris: float\\
beskrivelse: string\\
)
\end{minipage}
\vrule
\hspace{.03\textwidth}
\begin{minipage}[t]{.3\textwidth}
Stoffrull(\\
serienr: int\\
stofftype*: int\\
meter: float\\
)
\end{minipage}

# Oppgave 2)

1. Finn hvor mange (antallet) stolmodeller som finnes av hver stoltype.

    ```sql
    select
        stoltype.navn,
        count(standardstol.type) + count(spesialstol.type)
    from 
        standardstol, spesialstol
        natural join stoltype;
    ```

2. Ut fra alle registrerte ordre (bestillinger): Finn gjennomsnittlig antall bestilte stoler av hver stoltype. 

    ```sql
    select
        sum(antall)/count(ordre.ordrenr)
    from
        ordre_standardstol
    where
        ordre.ordrenr = ordre_standardstol.ordrenr
    group by type
    union
    select
        sum(antall)/count(ordre.ordrenr)
    from
        ordre, ordre_spesialstol 
    where
        ordre.ordrenr = ordre_spesialstol.ordrenr
    group by type;
    ```

3. Finn det totale antallet stoler som finnes i bestilling, og som enda ikke er levert kunder. Tips: Sjekk på reell leveringsdato (dvs. om ordren er effektuert). 

    ```sql
    select
        sum(ordre_standardstol.antall) + sum(ordre_spesialstol.antall)
    from
        ordre_standardstol, ordre_spesialstol, ordre
    where
        levert_dato is null;
    ```

4. Finn ut hvor mange (antallet) av stolene i spørring 3 over som er standardstoler.

    ```sql
    select
        sum(antall)
    from
        ordre_standardstol, ordre
    where
        levert_dato is null;
    ```
