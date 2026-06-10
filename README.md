# IS Šachové zápasy
Informační systém pro správu a organizaci šachových turnajů. Zaměřuje se na automatizaci nasazování hráčů (švýcarský systém, každý s každým), evidenci výsledků a generování konečného pořadí včetně pomocných hodnocení.

## Role v systému
* **Host:** Neregistrovaný návštěvník sledující veřejné výsledky a startovní listiny.
* **Hráč:** Registrovaný uživatel s profilem (ELO, FIDE ID) účastnící se turnajů.
* **Organizátor / Rozhodčí:** Zakládá turnaje, spravuje startovní listinu, řeší nasazení a spory, kontroluje zápasy.
* **Admin:** Kompletní správa systému, uživatelů a globálních definic.

## Odkazy na dokumentaci

- [UML Diagramy](diagramy/)
- Náhled UML: [index.html](https://pslib-cz.github.io/2025-p1a-inf-nis-JakubReif/)

## Katalog požadavků

# Katalog požadavků: IS Šachové zápasy

## 1. Aktéři (Role)
* **Nepřihlášený uživatel (Host)** – Může prohlížet veřejné turnaje, startovní listiny a výsledky.
* **Hráč** – Má účet, profil s ELO/FIDE ID. Přihlašuje se do turnajů, potvrzuje výsledky.
* **Organizátor / Rozhodčí** – Zakládá turnaje, spravuje zúčastněné, generuje párování (kola), řeší spory a kontumace.
* **Admin** – Správa celého systému, uživatelů a globálních číselníků.

## 2. Funkční požadavky (FR)

### Modul: Uživatelé a Autentizace
* **FR01: Registrace a profil hráče** – Vytvoření účtu (Ověření e-mailu). Profil obsahuje: Jméno, E-mail, Příslušnost ke klubu, Národní ELO, FIDE ID.
* **FR02: Autentizace a RBAC** – Login (e-mail + heslo), obnova hesla. Systém aplikuje práva na základě role.

### Modul: Správa turnajů (Pre-tournament)
* **FR03: Založení turnaje** – Organizátor definuje: Název, Datum, Kapacitu, Časové tempo (Standard/Rapid/Blitz), Systém (Švýcarský / Každý s každým), Počet kol, Pomocná hodnocení (např. Buchholz, Sonneborn-Berger).
* **FR04: Přihlášky a Startovní listina** – Hráč se může přihlásit. Organizátor schvaluje přihlášky, řeší seznam náhradníků a seřadí startovní listinu (typicky dle ELO).
* **FR05: Odstoupení z turnaje** – Hráč se může odhlásit před turnajem. Pokud odstoupí/je diskvalifikován v průběhu, systém ho vyřadí z losování dalších kol, ale zachová jeho dosavadní výsledky.

### Modul: Průběh turnaje (In-tournament)
* **FR06: Nasazení kola (Párování)** – Systém automaticky navrhne dvojice pro dané kolo. Zohledňuje: 
	* a) Zákaz opakovaného zápasu stejných hráčů.
	* b) Vyvažování barev figur (hráč by neměl mít 3x po sobě bílé).
	* c) Přidělení volného losu (Bye = 1 bod) při lichém počtu hráčů.
* **FR07: Manuální korekce losu** – Rozhodčí musí mít možnost systémový návrh párování před zveřejněním ručně upravit.
* **FR08: Zadání výsledku formátu zápasu** – Výstup zápasu má dedikované stavy: `1-0`, `0-1`, `½-½`, `+ -` (kontumace bílý), `- +` (kontumace černý), `0-0` (oboustranná kontumace).
* **FR09: Potvrzení výsledku** – Výsledek může zadat hráč, ale pak jej musí potvrdit druhý hráč nebo Rozhodčí. Dokud nejsou potvrzeny všechny zápasy, nejde vylosovat další kolo.
* **FR10: Detail zápasu (Protesty)** – Možnost k zápasu přidat textovou poznámku (např. nahlášení nepřípustného tahu).

### Modul: Výsledky a Export (Post-tournament)
* **FR11: Průběžné a konečné pořadí** – Systém po každém kole přepočítává tabulku. Skóre primárně dle bodů, při shodě se aplikují zvolená pomocná hodnocení z FR03.
* **FR12: Notifikace** – Zveřejnění nového kola zašle registrovaným hráčům oznámení (číslo stolu a barva figur).
* **FR13: Export výsledků a reporty** – Možnost exportovat tabulku a výsledky kol (např. formát CSV pro report na šachový svaz / PGN pro partiový záznam).

## 3. Nefunkční požadavky (NFR)
* **NFR01: Mobilní přívětivost (Responsive UI)** – UI musí být perfektně čitelné na telefonech, protože hráči na turnaji kontrolují losování přes mobily v reálném čase.
* **NFR02: Výkonnost u losování kol** – Algoritmus pro nasazení (zejména Švýcarský systém u 100+ hráčů) musí vygenerovat návrh do 3 sekund.
* **NFR03: Ochrana proti souběžným zápisům** – Zámek databáze při ukládání výsledků (zabraňuje tomu, aby konflikt dvou žádostí přepsal již zadaný výsledek).

## UML a diagramy
- UML diagramy jsou v adresáři `diagramy/` a pokrývají role, funkce a akce, data a vztahy i technologie a nasazení.
- Náhled je v [index.html](index.html).