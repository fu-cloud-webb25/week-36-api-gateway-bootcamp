# Vecka 36: API Gateway Bootcamp - veckans Code Review-uppgift

I denna övning ska du bygga ett mindre REST-liknande API med hjälp av **AWS Lambda, API Gateway och Serverless Framework**.

Vi kommer steg för steg bygga ett **Shakespearean Insults API** där användaren kan hämta, lägga till, ändra och ta bort förolämpningar från Shakespeares pjäser.

Varje endpoint ska kopplas till en egen Lambda-funktion.

När du är klar kommer ditt API bland annat innehålla följande endpoints:

```text
GET    /insults
GET    /insults/{id}
POST   /insults
PUT    /insults/{id}
DELETE /insults/{id}
```

---

# Startdata

Skapa en fil för din data och lägg in följande array:

```js
const insults = [
  {
    id: 1,
    insult: "Peace! I will not endure thy lies.",
    play: "Much Ado About Nothing"
  },
  {
    id: 2,
    insult: "Thy tongue outvenoms all the worms of Nile.",
    play: "Cymbeline"
  },
  {
    id: 3,
    insult: "Thou crusty batch of nature!",
    play: "Troilus and Cressida"
  },
  {
    id: 4,
    insult: "Thou art a boil, a plague sore!",
    play: "King Lear"
  },
  {
    id: 5,
    insult: "Away, you three-inch fool!",
    play: "Taming of the Shrew"
  },
  {
    id: 6,
    insult: "Thou poisonous bunch-backed toad!",
    play: "Richard III"
  }
];

export default insults;
```

> **OBS!** Vi använder en vanlig array för att kunna fokusera på API Gateway och Lambda. Våra Lambda-funktioner har ingen permanent lagring, vilket innebär att ändringar i arrayen inte kan förväntas finnas kvar mellan olika anrop. Senare kommer vi lösa detta genom att koppla våra funktioner till en databas.

---

# Steg 1 – Hämta alla insults

Vi börjar enkelt.

Skapa en Lambda-funktion som returnerar samtliga insults.

Endpointen ska vara:

```http
GET /insults
```

Funktionen ska returnera:

```json
{
  "statusCode": 200,
  "body": [
    {
      "id": 1,
      "insult": "Peace! I will not endure thy lies.",
      "play": "Much Ado About Nothing"
    }
  ]
}
```

Arrayen ovan är förkortad i exemplet. Ditt API ska självklart returnera alla insults.

### Serverless Framework

Skapa en funktion i `serverless.yml` och koppla den till ett HTTP API-event:

```yaml
functions:
  getInsults:
    handler: functions/getInsults.handler
    events:
      - httpApi:
          path: /insults
          method: get
```

Deploya sedan applikationen:

```bash
serverless deploy
```

Testa endpointen med exempelvis Postman eller Insomnia.

---

# Steg 2 – Hämta en specifik insult

Nu ska användaren kunna hämta en specifik insult.

Vi vill exempelvis kunna anropa:

```http
GET /insults/3
```

och få tillbaka:

```json
{
  "id": 3,
  "insult": "Thou crusty batch of nature!",
  "play": "Troilus and Cressida"
}
```

För detta ska vi använda en **path parameter**.

Konfigurera routen:

```yaml
path: /insults/{id}
method: get
```

Undersök sedan `event` i din Lambda-funktion.

Hur kommer du åt värdet som skickats in istället för `{id}`?

### Krav

Om användaren anropar:

```http
GET /insults/3
```

ska insult med `id: 3` returneras.

Om ID:t inte finns ska API:t istället returnera:

```json
{
  "message": "Insult not found."
}
```

med statuskod:

```text
404 Not Found
```

---

# Steg 3 – Lägg till en insult

Nu ska API:t kunna ta emot data.

Skapa endpointen:

```http
POST /insults
```

Användaren ska skicka följande JSON i requestens body:

```json
{
  "insult": "Thou art as fat as butter.",
  "play": "Henry IV"
}
```

Skapa en Lambda-funktion som:

1. Läser requestens `body`.
2. Gör om bodyn från JSON till ett JavaScript-objekt.
3. Skapar ett nytt ID.
4. Lägger till objektet i arrayen.
5. Returnerar det skapade objektet.

Ett lyckat anrop ska returnera:

```text
201 Created
```

och exempelvis:

```json
{
  "id": 7,
  "insult": "Thou art as fat as butter.",
  "play": "Henry IV"
}
```

> Kom ihåg att `event.body` behöver parsas innan du kan arbeta med innehållet som ett vanligt JavaScript-objekt.

---

# Steg 4 – Ersätt en insult

Nu ska användaren kunna ersätta en befintlig insult.

Skapa endpointen:

```http
PUT /insults/{id}
```

Exempel:

```http
PUT /insults/4
```

Requestens body kan exempelvis innehålla:

```json
{
  "insult": "Villain, I have done thy mother!",
  "play": "Titus Andronicus"
}
```

Din funktion ska:

1. Hämta ID:t från path-parametern.
2. Hämta den nya datan från requestens body.
3. Leta efter motsvarande insult i arrayen.
4. Ersätta den gamla informationen.
5. Returnera det uppdaterade objektet.

Om ingen insult med angivet ID finns ska:

```text
404 Not Found
```

returneras.

---

# Steg 5 – Ta bort en insult

Sista delen av vår CRUD-funktionalitet är `DELETE`.

Skapa endpointen:

```http
DELETE /insults/{id}
```

Exempel:

```http
DELETE /insults/3
```

Funktionen ska:

1. Hämta ID:t från path-parametern.
2. Leta efter motsvarande insult.
3. Ta bort den ur arrayen.
4. Returnera ett lämpligt svar.

Om förolämpningen inte finns ska API:t returnera:

```text
404 Not Found
```

---

# Kontrollera ditt API

Du bör nu ha byggt följande endpoints:

| Method   | Endpoint        | Funktion   |
| -------- | --------------- | ---------- |
| `GET`    | `/insults`      | Hämta alla |
| `GET`    | `/insults/{id}` | Hämta en   |
| `POST`   | `/insults`      | Skapa en   |
| `PUT`    | `/insults/{id}` | Ersätt en  |
| `DELETE` | `/insults/{id}` | Ta bort en |

Varje endpoint ska vara kopplad till en **egen Lambda-funktion**.

Testa samtliga endpoints med Postman eller Insomnia innan du går vidare.

---

# Steg 6 – Query Parameters

Nu har vi ett fungerande API. Dags att bygga ut det med filtrering.

Vi vill fortfarande använda:

```http
GET /insults
```

men användaren ska kunna skicka med olika **query parameters**.

## Sök efter text

Använd query-parametern `search` för att söka efter text i förolämpningarna.

Exempel:

```http
GET /insults?search=toad
```

ska returnera alla insults vars `insult` innehåller `toad`.

Sökningen bör inte vara case-sensitive.

Det betyder att:

```text
toad
TOAD
Toad
```

ska ge samma resultat.

---

## Sök efter pjäs

Använd query-parametern `play` för att filtrera efter vilken pjäs förolämpningen kommer från.

Exempel:

```http
GET /insults?play=King%20Lear
```

ska endast returnera insults från *King Lear*.

---

## Kombinera query parameters

### Level Up!

Låt användaren kombinera båda parametrarna:

```http
GET /insults?play=King%20Lear&search=boil
```

Resultatet ska då uppfylla **båda** villkoren.

---

# Steg 7 – Validering

### Level Up!

Gör API:t lite mer robust.

När någon försöker skapa eller ersätta en insult ska både:

```text
insult
play
```

finnas med.

Följande request:

```json
{
  "insult": "Thou art terrible!"
}
```

saknar `play` och ska därför returnera:

```text
400 Bad Request
```

med exempelvis:

```json
{
  "message": "Insult and play are required."
}
```

---

# Färdig!

Du har nu byggt ett API med:

* API Gateway
* Serverless Framework
* flera Lambda-funktioner
* GET
* POST
* PUT
* DELETE
* path parameters
* query parameters
* request body
* HTTP-statuskoder
* enkel validering

Ta en titt på din `serverless.yml`.

Du har gått från en vanlig JavaScript-funktion till flera Lambda-funktioner som exponeras genom ett gemensamt HTTP API.

---

# Del 2 – Bygg ditt eget API

Nu har du byggt ett komplett API steg för steg.

Dags att bygga ett eget!

Skapa ett **nytt Serverless Framework-projekt** och bygg ett API med ett tema som du själv väljer.

Exempel:

* Film-API
* Bok-API
* Pokémon-API
* Recept-API
* Rese-API
* Event-API
* Spel-API
* Musik-API
* Sport-API
* Rollspels-API

Eller hitta på något helt eget.

## Krav

Ditt API ska innehålla funktionalitet för:

* `GET`
* `POST`
* `PUT`
* `DELETE`
* minst en endpoint med en **path parameter**
* minst en endpoint som använder en **query parameter**
* request body för att skicka data
* relevanta HTTP-statuskoder

Precis som tidigare ska olika endpoints hanteras av separata Lambda-funktioner.

Du ska själv bestämma:

* vilken data ditt API innehåller
* vilka endpoints som behövs
* vad användaren kan göra
* hur dina objekt ska se ut
* vilka query parameters som är användbara

> Tänk på att dina Lambda-funktioner fortfarande saknar permanent lagring. Använd därför en grundarray med startdata. Vi kommer senare ersätta detta med riktig datalagring.

---

# Level Up!

När grundkraven fungerar kan du bygga vidare på ditt API.

Försök skapa funktionalitet som faktiskt passar temat du valt.

Har du byggt ett bok-API?

Kanske användaren kan köpa böcker eller filtrera på författare och genre.

Har du byggt ett Pokémon-API?

Kanske två Pokémon kan slåss mot varandra.

Har du byggt ett recept-API?

Kanske användaren kan ange ingredienser och få förslag på recept.

Har du byggt ett event-API?

Kanske användaren kan boka eller avboka biljetter.

Har du byggt ett rollspels-API?

Kanske API:t kan generera en slumpmässig karaktär.

**Möjligheterna är oändliga – försök göra API:t till ditt eget.**
