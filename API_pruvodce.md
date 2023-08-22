<img height="30" src="logo.svg" title="Keyguru logo" width="200"/>

# Keyguru API průvodce

Verze API: **2.2.1**<br>
Datum vydání API: **2023-08-22**<br>
[Webové rozhraní Swagger](https://keyguru.app/api/ui/)

## Změny

### 2023-08-22 / API verze 2.2.1

* Časová razítka v **[historii zařízení](#post-device-identification-history)** nyní vyjadřují okamžik, kdy událost
  nastala, namísto okamžiku, kdy byla událost přijata na serveru.

### 2022-12-01 / API verze 2.2.0

* Nový endpoint "[**PATCH drawer reservation**](#patch-drawer-reservation)", který umožňuje **úplnou změnu rezervace**.

## Úvod

Adresa API serveru je `https://keyguru.app/api/` doplněno o [konkrétní endpoint](#api-endpointy).
Například `https://keyguru.app/api/v1/self`

Keyguru API je založeno na [OpenAPI 3.0.0](https://swagger.io/specification/v3/). Každý požadavek musí být autorizován
pomocí API klíče v hlavičce `X-Api-Key`. Pro získání API klíče se na nás oobraťe na
[helpdesk@keyguru.cz](mailto:helpdesk@keyguru.cz).

Funkci API můžete testovat pomocí [webového rozhraní Swagger](https://keyguru.app/api/ui/), které poskytuje stručný
popis jednotlivých [endpointů](#api-endpointy). Tento průvodce poskytuje širší souvislosti.

Swagger můžete také použít k automatickému vygenerování částí zdrojového kódu pro integraci s naším API:
Stáhněte si náš [definiční soubor](https://keyguru.app/api/ui/index.yaml) a vložte jeho obsah do levého sloupce
[Swagger editoru](https://editor.swagger.io/). V horním menu editoru je sekce "Generate Client".

### Jak fungují rezervace

**Rezervace** = časové rozmezí, ve kterém je přihrádka v boxu přístupná pro hosta. Rezervace přihrádky může odpovídat
rezervaci pokoje nebo se může lišit.

#### Varianty mapování

Příklad: Host si zarezervoval pokoj na 3 noci. Očekávaný příjezd je ve 2:00 ráno.

##### 1. Každý pokoj má svou přihrádku

- Pokoj i přihrádka jsou rezervované na 3 noci. Host si může kdykoliv vyzvednout nebo vrátit klíče a to i opakovaně.

- **Výhody**: Jednoduché a plně bezobslužné.

- **Nevýhody**: Vyžaduje to tolik přihrádek, kolik je pokojů. Neefektivní zejména v období malého provozu.

##### 2. Přihrádky pouze pro předání klíčů

- Přihrádka je rezervovaná pouze na noc příjezdu hosta. Vrácení klíčů probíhá mimo box. Případně se vytvoří další
  rezervace pouze na den odjezdu hosta.

- **Výhody:** Efektivnější využití přihrádek. Počet přihrádek může být výrazně menší, než počet pokojů.

- **Nevýhody:** Postup je složitější, klíče se vrací na recepci nebo zůstávají ve dveřích pokoje.

### Základní pojmy

#### _X-Api-Key_ = API klíč zákazníka

- API vyžaduje autorizaci každého požadavku pomocí klíče v hlavičce `X-Api-Key`.

#### _Zařízení_ = přístupový systém zákazníka

- Obvykle 1 hotel = 1 přístupový systém.
- Přístupový systém se skládá z jednotlivých boxů a gatekeeperů, které však není nutné adresovat v API požadavcích.

#### _Přihrádka_ = přihrádka v boxu

- Nejmenší jednotka v přístupovém systému zákazníka
- Rezervace je vždy přiřazena k přihrádce

#### _Kód hosta_ = přístupový kód

- Přístupový kód hosta platí po dobu rezervace přihrádky.
- Každá rezervace má 1 přístupový kód hosta.
- Kód hosta musí být unikátní v rámci přístupového systému. Různé rezervace mohou mít stejný kód pouze tehdy, když se
  platnost těchto rezervací nepřekrývá na časové ose.
- Kód hosta může být buď zadán nebo vygenerován automaticky.
- Personál využívá své vlastní přístupové kódy nezávisle na rezervacích. Přístupové kódy pro personál nelze vytvářet
  ani měnit přes API ale pouze přes webové administrační rozhraní.

### Uživatelské názvy

_Zařízení_ a _příhrádky_ můžou mít uživatelské názvy. Lze je adresovat pomocí jejich názvu nebo ID.

- Název _zařízení_ musí být unikátní v rámci zákazníka.
- Název _přihrádky_ musí být unikátní v rámci _zařízení_.

Název rezervace **nemusí** být unikátní (např. "Novákovi"). Rezervaci lze proto adresovat pouze pomocí jejího ID.

### Formát data a času

Hodnoty data a času by měly odpovídat normě [ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html).
Je podporován formát UTC (Z) i offset (např. +02:00).

- `2021-06-07T14:18:19Z`
- `2021-06-07T14:18:19.123Z`
- `2021-06-07T14:18:19.123456Z`
- `2021-06-07T16:18:19+02:00`

API odpovídá ve formátu UTC (Z).

Každému _zařízení_ můžete nastavit časové pásmo ve webovém administračním rozhraní.

### Autorizace a řízení přístupu

API klíč zákazníka umožňuje přístup do všech zařízení daného zákazníka.

### Duplikace některých GET endpointů na POST

V souladu s [RFC 7231](https://www.rfc-editor.org/rfc/rfc7231#section-4.3.1) mnoho HTTP klientů dnes nepodporuje tělo
v GET požadavcích. Proto jsme některé GET endpointy museli duplikovat na POST. Původní GET endpointy budou zachovány
kvůli zpětné kompatibilitě.

## API endpointy

### [GET self](https://keyguru.app/api/ui/#/default/get_v1_self)

`/v1/self`

Tato funkce pouze **ověřuje platnost API klíče** předávaného v hlavičce `X-Api-Key` a vrací základní údaje o
zákazníkovi.

### [GET device](https://keyguru.app/api/ui/#/default/get_v1_device)

`/v1/device`

Tato funkce vrací **všechna zařízení zákazníka**.

### [GET device {identification}](https://keyguru.app/api/ui/#/default/get_v1_device__deviceIdentification_)

`/v1/device/{deviceIdentification}`

Tato funkce vrací **konkrétní zařízení zákazníka**.

**Povinné parametry:**

- název nebo ID zařízení

### [PATCH device {identification}](https://keyguru.app/api/ui/#/default/patch_v1_device__deviceIdentification_)

`/v1/device/{deviceIdentification}`

Tato funkce prozatím umožňuje pouze **změnu názvu zařízení**. Název musí být jedinečný v rámci všech zařízení zákazníka.

**Povinné parametry:**

- název nebo ID zařízení
- nový název zařízení

### [POST device {identification} history](https://keyguru.app/api/ui/#/default/post_v1_device__deviceIdentification__history)

`/v1/device/{deviceIdentification}/history`

Tato funkce vrací **historii zařízení**. Vypíše všechny události spojené s konkrétním zařízením:

- události ze zařízení (např. "open_drawer")
- události vyvolané obsluhou (např. "reservation_create")
- události vyvolané technickým personálem Keyguru (např. "device_assign")
- události vyvolané přes API

**Povinné parametry:**

- název nebo ID zařízení

**Volitelné parametry:**

- omezení časového rozmezí (výchozí: všechny události)
- omezení počtu položek (výchozí: 1000)

### [GET drawer](https://keyguru.app/api/ui/#/default/get_v1_device__deviceIdentification__drawer)

`/v1/device/{deviceIdentification}/drawer`

Tato funkce vrací **všechny přihrádky** v konkrétním zařízení zákazníka.

**Povinné parametry:**

- název nebo ID zařízení

### [GET drawer {identification}](https://keyguru.app/api/ui/#/default/get_v1_device__deviceIdentification__drawer__drawerIdentification_)

`v1/device/{deviceIdentification}/drawer/{drawerIdentification}`

Tato funkce vrací **konkrétní přihrádku** v zařízení zákazníka.

**Povinné parametry:**

- název nebo ID zařízení
- název nebo ID přihrádky

### [PATCH drawer {identification}](https://keyguru.app/api/ui/#/default/patch_v1_device__deviceIdentification__drawer__drawerIdentification_)

`v1/device/{deviceIdentification}/drawer/{drawerIdentification}`

Tato funkce v současnosti umožňuje pouze **změnu názvu přihrádky**. Název musí být jedinečný v rámci zařízení.

**Povinné parametry:**

- název nebo ID zařízení
- název nebo ID přihrádky
- nový název přihrádky

### [GET device reservation](https://keyguru.app/api/ui/#/default/get_v1_device__deviceIdentification__reservation)

`/v1/device/{deviceIdentification}/reservation`

Tato funkce vrací **všechny rezervace v zařízení kromě smazaných** rezervací.

**Povinné parametry:**

- název nebo ID zařízení

Tato funkce je zachována kvůli zpětné kompatibilitě. Doporučujeme používat novější funkci
[**POST device reservation-list**](#post-device-reservation-list), která navíc umožňuje **filtrování** a **zobrazení
smazaných rezervací**.

Jestli potřebujete rezervace pouze pro **konkrétní přihrádku**, můžete použít jednu z následujících funkcí:

* [**POST drawer reservation-list**](#post-drawer-reservation-list)
    * **doporučeno**
    * umožňuje filtrování a zobrazení smazaných rezervací
* [**GET drawer reservation**](#get-drawer-reservation)
    * **zachováno kvůli zpětné kompatibilitě**
    * neumožňuje filtrování ani zobrazení smazaných rezervací

### [POST device reservation](https://keyguru.app/api/ui/#/default/post_v1_device__deviceIdentification__reservation)

`/v1/device/{deviceIdentification}/reservation`

Tato funkce **vytvoří a vrátí novou rezervaci. Přihrádka bude k rezervaci přidělena automaticky.** Rezervace v rámci
jedné přihrádky se nesmí překrývat. Pokud není k dispozici žádná volná přihrádka, funkce vrátí chybu.

**Povinné parametry:**

- název nebo ID zařízení
- název rezervace
- začátek platnosti rezervace
- konec platnosti rezervace
    - minimální trvání: 1 hodina

**Volitelné parametry:**

- přístupový kód hosta
    - 5 místné číslo
    - musí být unikátní v rámci všech aktuálně platných rezervací v zařízení, jinak funkce vrátí chybu
    - pokud není obsažen v požadavku, bude vygenerován automaticky
- poznámka k rezervaci

**UPOZORNĚNÍ:** Přístupový kód je potřeba následně sdělit hostovi např. emailem nebo SMS zprávou.

Jestli potřebujete vytvořit novou rezervaci v **konkrétní přihrádkce**, použijte funkci
[**POST drawer reservation**](#post-drawer-reservation).

### [POST device reservation-list](https://keyguru.app/api/ui/#/default/post_v1_device__deviceIdentification__reservation_list)

`/v1/device/{deviceIdentification}/reservation-list`

Tato funkce vrací **všechny rezervace v zařízení** podobně, jako funkce
[**GET device reservation**](#get-device-resevation) s tím rozdílem, že tato funkce navíc umožňuje **filtrování** a
**zobrazení smazaných rezervací**.

**Povinné parametry:**

- název nebo ID zařízení

**Volitelné parametry:**

- časové rozmezí
    - od
    - do
- max. počet rezervací (výchozí: 1000)
- zahrnout i smazané rezervace (vychozí: `false`)

Jestli potřebujete rezervace pouze pro **konkrétní přihrádku**, můžete použít jednu z následujících funkcí:

* [**POST drawer reservation-list**](#post-drawer-reservation-list)
    * **doporučeno**
    * umožňuje filtrování a zobrazení smazaných rezervací
* [**GET drawer reservation**](#get-drawer-reservation)
    * **zachováno kvůli zpětné kompatibilitě**
    * neumožňuje filtrování ani zobrazení smazaných rezervací

### [GET drawer reservation](https://keyguru.app/api/ui/#/default/get_v1_device__deviceIdentification__drawer__drawerIdentification__reservation)

`/v1/device/{deviceIdentification}/drawer/{drawerIdentification}/reservation`

Tato funkce vrací **všechny rezervace v konkrétní přihrádce** v zařízení **kromě smazaných rezervací**.

**Povinné parametry:**

- název nebo ID zařízení
- název nebo ID přihrádky

Tato funkce je zachována kvůli zpětné kompatibilitě. Doporučujeme používat novější funkci
[**POST drawer reservation-list**](#post-drawer-reservation-list), která navíc umožňuje **filtrování** a **zobrazení
smazaných rezervací**.

Jestli potřebujete **všechny rezervace z celého zařízení**, můžete použít jednu z následujících funkcí:

* [**POST device reservation-list**](#post-device-reservation-list)
    * **doporučeno**
    * umožňuje filtrování a zobrazení smazaných rezervací
* [**GET device reservation**](#get-device-reservation)
    * **zachováno kvůli zpětné kompatibilitě**
    * neumožňuje filtrování ani zobrazení smazaných rezervací

### [POST drawer reservation](https://keyguru.app/api/ui/#/default/post_v1_device__deviceIdentification__drawer__drawerIdentification__reservation)

`/v1/device/{deviceIdentification}/drawer/{drawerIdentification}/reservation`

Tato funkce **vytvoří a vrátí novou rezervaci pro konkrétní přihrádku** v zařízení. Rezervace v rámci jedné přihrádky se
nesmí překrývat. Pokud přihrádka není volná, funkce vrátí chybu.

**Povinné parametry:**

- název nebo ID zařízení
- název nebo ID přihrádky
- název rezervace
- začátek platnosti rezervace
- konec platnosti rezervace
    - minimální trvání: 1 hodina

**Volitelné parametry:**

- přístupový kód hosta
    - 5 místné číslo
    - musí být unikátní v rámci všech aktuálně platných rezervací v zařízení, jinak funkce vrátí chybu
    - pokud není obsažen v požadavku, bude vygenerován automaticky
- poznámka k rezervaci

**UPOZORNĚNÍ:** Přístupový kód je potřeba následně sdělit hostovi např. emailem nebo SMS zprávou.

Jestli potřebujete **automaticky vyhledat volnou přihrádku v rámci zařízení**, použijte funkci
[**POST device reservation**](#post-device-reservation).

### [POST drawer reservation-list](https://keyguru.app/api/ui/#/default/post_v1_device__deviceIdentification__drawer__drawerIdentification__reservation_list)

`/v1/device/{deviceIdentification}/drawer/{drawerIdentification}/reservation-list`

Tato funkce vrací **všechny rezervace v konkrétní přihrádce** zařízení podobně, jako funkce
[**GET drawer reservation**](#get-drawer-resevation) s tím rozdílem, že tato funkce navíc umožňuje **filtrování** a
**zobrazení smazaných rezervací**.

**Povinné parametry:**

- název nebo ID zařízení
- název nebo ID přihrádky

**Volitelné parametry:**

- časové rozmezí
    - od
    - do
- max. počet rezervací (výchozí: 1000)
- zahrnout i smazané rezervace (vychozí: `false`)

Jestli potřebujete **všechny rezervace z celého zařízení**, můžete použít jednu z následujících funkcí:

* [**POST device reservation-list**](#post-device-reservation-list)
    * **doporučeno**
    * umožňuje filtrování a zobrazení smazaných rezervací
* [**GET device reservation**](#get-device-reservation)
    * **zachováno kvůli zpětné kompatibilitě**
    * neumožňuje filtrování ani zobrazení smazaných rezervací

### [PATCH drawer reservation](https://keyguru.app/api/ui/#/default/patch_v1_device__deviceIdentification__drawer__drawerIdentification__reservation__reservationId_)

`/v1/device/{deviceIdentification}/drawer/{drawerIdentification}/reservation/{reservationId}`

Tato funkce umožňuje úplnou **změnu existující rezervace**. Jeden nebo více atributů může být změněno současně.

**Povinné parametry:**

- název nebo ID zařízení
- název nebo ID přihrádky
- ID rezervace

**Volitelné parametry:**

- nový název rezervace
- nový přístupový kód hosta
    - 5 místné číslo
    - musí být unikátní v rámci všech aktuálně platných rezervací v zařízení, jinak funkce vrátí chybu
    - pokud kód není obsažen v požadavku, **nebude změněn**
- název nebo ID nové přihrádky
- nový začátek platnosti rezervace
- nový konec platnosti rezervace
    - minimální trvání: 1 hodina
- nová poznámka k rezervaci

### [DELETE reservation](https://keyguru.app/api/ui/#/default/delete_v1_device__deviceIdentification__drawer__drawerIdentification__reservation__reservationId_)

`/v1/device/{deviceIdentification}/drawer/{drawerIdentification}/reservation/{reservationId}`

Tato funkce **smaže existující rezervaci**.

**Povinné parametry:**

- název nebo ID zařízení
- název nebo ID přihrádky
- ID rezervace

Jestli potřebujete existující rezervaci **změnit nebo přesunout do jiné přihrádky**, použijte funkci
[**PATCH drawer reservation**](#patch-drawer-reservation).

### [POST new-code](https://keyguru.app/api/ui/#/default/post_v1_device__deviceIdentification__drawer__drawerIdentification__reservation__reservationId__new_code)

`/v1/device/{deviceIdentification}/drawer/{drawerIdentification}/reservation/{reservationId}/new-code`

Tato funkce **změní přístupový kód** existující rezervace a vrátí nový kód.

**Povinné parametry:**

- název nebo ID zařízení
- název nebo ID přihrádky
- ID rezervace

**Volitelné parametry:**

- nový přístupový kód hosta
    - 5 místné číslo
    - musí být unikátní v rámci všech aktuálně platných rezervací v zařízení, jinak funkce vrátí chybu
    - pokud není obsažen v požadavku, bude **vygenerován náhodný kód**
