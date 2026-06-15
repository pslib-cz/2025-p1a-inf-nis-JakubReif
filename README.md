# IS Šachové zápasy

Informační systém pro správu a organizaci šachových turnajů. Zaměřuje se na automatizaci nasazování hráčů (švýcarský systém, každý s každým), evidenci výsledků a generování konečného pořadí včetně pomocných hodnocení.

## Architektura systému

Systém je rozdělen do tří vrstev:

- **Frontend (React SPA)** – responzivní webová aplikace pro hráče a organizátory, odesílá REST požadavky na backend.
- **Backend (REST API + Services)** – Node.js/Express server implementující veškerou business logiku, autentizaci, párování kol a export.
- **Databáze (PostgreSQL / Supabase)** – relační úložiště doménového modelu s ochranou před souběžnými zápisy.

## Role v systému

Role tvoří hierarchii dědičnosti – každá role zahrnuje veškerá oprávnění rolí pod ní:

| Role | Dědí od | Popis |
|---|---|---|
| **Host** | – | Neregistrovaný návštěvník; sleduje veřejné výsledky a startovní listiny. |
| **Hráč** | Host | Registrovaný uživatel s profilem (ELO, FIDE ID); účastní se turnajů, zadává a potvrzuje výsledky. |
| **Organizátor** | Hráč | Zakládá turnaje, spravuje startovní listinu, generuje a publikuje kola, exportuje výsledky. |
| **Rozhodčí** | Organizátor | Manuálně koriguje párování, řeší protesty a kontumace, uzavírá kola. |
| **Admin** | Rozhodčí | Kompletní správa systému, uživatelů, rolí a globálních číselníků. |

## UML Diagramy

Diagramy jsou ve formátu PlantUML (`.puml`) v adresáři `diagramy/`.

| Soubor | Typ | Obsah |
|---|---|---|
| `uml-role.puml` | Use Case | Role, dědičnost, případy užití navázané na FR |
| `uml-data.puml` | Class | Doménový model (backend), services a DTO/view modely (frontend) |
| `uml-akce.puml` | Sequence | Životní cyklus turnaje – předávání zpráv mezi aktéry a systémem |
| `uml-nasazeni.puml` | Deployment | Architektura nasazení: Frontend / Backend / DB + NFR vazby |

Náhled: [index.html](https://pslib-cz.github.io/2025-p1a-inf-nis-JakubReif/)

---

# Katalog požadavků: IS Šachové zápasy

## 1. Aktéři (Role)

Role tvoří dědičnostní hierarchii (každá vyšší role zahrnuje oprávnění nižších):

- **Host (Nepřihlášený uživatel)** – Může prohlížet veřejné turnaje, startovní listiny a výsledky.
- **Hráč** *(dědí od Host)* – Má účet a profil (ELO, FIDE ID, klub). Přihlašuje se do turnajů, zadává a potvrzuje výsledky, podává protesty.
- **Organizátor** *(dědí od Hráč)* – Zakládá turnaje, spravuje startovní listinu, generuje a publikuje párování kol, exportuje výsledky.
- **Rozhodčí** *(dědí od Organizátor)* – Manuálně koriguje systémový návrh párování, řeší spory, protesty a kontumace, uzavírá kola a turnaj.
- **Admin** *(dědí od Rozhodčí)* – Správa celého systému, uživatelů, rolí a globálních číselníků.

## 2. Funkční požadavky (FR)

### Modul: Uživatelé a Autentizace

- **FR01: Registrace a profil hráče** – Vytvoření účtu s ověřením e-mailem. Profil obsahuje: Jméno, E-mail, Příslušnost ke klubu, Národní ELO, FIDE ID (nepovinné).
- **FR02: Autentizace a RBAC** – Přihlášení (e-mail + heslo), obnova hesla. Systém přiděluje a vynucuje práva na základě role (Host → Hráč → Organizátor → Rozhodčí → Admin).

### Modul: Správa turnajů – Pre-tournament

- **FR03: Založení turnaje** – Organizátor definuje: Název, Datum, Kapacitu, Časové tempo (Standard / Rapid / Blitz), Systém (Švýcarský / Každý s každým), Počet kol, Pomocná hodnocení (Buchholz, Sonneborn-Berger, počet výher, přímé vzájemné zápasy).
- **FR04: Přihlášky a Startovní listina** – Hráč se přihlásí; Organizátor schvaluje žádosti, spravuje náhradníky a seřadí startovní listinu (typicky dle ELO).
- **FR05: Odstoupení z turnaje** – Hráč se může odhlásit před turnajem. Odstoupí-li nebo je-li diskvalifikován v průběhu, systém ho vyřadí z dalšího losování, ale zachová jeho dosavadní výsledky.

### Modul: Průběh turnaje – In-tournament

- **FR06: Nasazení kola (Párování)** – Systém automaticky navrhne dvojice pro kolo. Zohledňuje:
  - a) Zákaz opakovaného zápasu stejných hráčů.
  - b) Vyvažování barev figur (hráč nesmí mít 3× po sobě stejnou barvu).
  - c) Přidělení volného losu (Bye = 1 bod) při lichém počtu hráčů.
- **FR07: Manuální korekce losu** – Rozhodčí může systémový návrh párování před zveřejněním ručně upravit.
- **FR08: Zadání výsledku zápasu** – Výsledek má dedikované stavy: `1-0`, `0-1`, `½-½`, `+ -` (kontumace bílý výherce), `- +` (kontumace černý výherce), `0-0` (oboustranná kontumace).
- **FR09: Potvrzení výsledku** – Výsledek může zadat hráč, ale pak jej musí potvrdit druhý hráč nebo Rozhodčí. Dokud nejsou potvrzeny všechny zápasy kola, nelze vygenerovat kolo další.
- **FR10: Detail zápasu a protesty** – K zápasu lze přidat textovou poznámku (nahlášení protestu); Rozhodčí je notifikován a může výsledek přepsat.

### Modul: Výsledky a Export – Post-tournament

- **FR11: Průběžné a konečné pořadí** – Po každém kole systém přepočítá tabulku. Skóre primárně dle bodů; při shodě se aplikují pomocná hodnocení zvolená v FR03. Po uzavření turnaje proběhne finální přepočet.
- **FR12: Notifikace** – Zveřejnění nového kola rozešle registrovaným hráčům oznámení (číslo stolu a barva figur) e-mailem nebo SSE push notifikací.
- **FR13: Export výsledků a reporty** – Export tabulky a výsledků kol ve formátu CSV (report pro šachový svaz) a PGN (záznamy partií).

## 3. Nefunkční požadavky (NFR)

- **NFR01: Mobilní přívětivost (Responsive UI)** – Rozhraní musí být čitelné na telefonech, protože hráči sledují losování v reálném čase přímo na turnaji.
- **NFR02: Výkonnost losování kol** – Algoritmus pro nasazení musí zvládnout vygenerovat párování i pro větší počet hráčů bez zbytečného čekání.
- **NFR03: Ochrana proti souběžným zápisům** – Systém musí zajistit, aby dva souběžné požadavky nemohly přepsat stejný výsledek zápasu.