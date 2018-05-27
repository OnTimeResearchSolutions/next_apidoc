---
title: Puhelinkysely.fi API-rajapintakuvaus

language_tabs: # must be one of https://git.io/vQNgJ
  - php
  - shell

toc_footers:
  - <a href='https://github.com/lord/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---

# Johdanto

Tervetuloa [puhelinkysely.fi-palvelun](https://puhelinkysely.fi/) API-rajapintakuvaukseen. Tämän rajapinnan kautta voit tällä hetkellä lähettää ennalta luotuja kyselyitä soitettavaksi ja tulevaisuudessa hakea kyselyiden vastaustietoja ja soittaa ad hoc -kyselyitä.

Koodiesimerkit on esitetty PHP-kielellä sekä komentorivikäskyinä, mutta REST-tyyppinen rajapinta on käytettävissä myös muilla kielillä ja skripteillä oman tarpeesi mukaisesti. Tällä hetkellä emme tarjoa valmiita kielikohtaisia API-kirjastoja, vaan rajapinnan käytön toteutus on omalla vastuullasi.

# Autentikointi

> API-tunnistautuminen jokaisen pyynnön yhteydessä:

```php
<?php

$apiKey = 'APIAVAIN';
$apiSecret = 'SALAISUUS';
$apiNonce = 123;

$payload = new stdClass(); // määrittele pyyntösi data
$requestData = json_encode($payload);

$signature = hash_hmac("sha256", $requestData . $apiNonce, $apiSecret);

// lähetä pyyntö
$options = array(
    'http' => array(
        'header'  => "Content-Type: application/json\r\n"
                   . "Authorization: APIAVAIN\r\n"
                   . "X-Phonepoll-Nonce: {$apiNonce}\r\n"
                   . "X-Phonepoll-Signature: {$signature}\r\n",
        'method'  => 'POST',
        'content' => $requestData
    )
);
$context  = stream_context_create($options);
$result = file_get_contents("https://puhelinkysely.fi/api/v1/call", false, $context);

?>
```

```shell
curl 'https://puhelinkysely.fi/api/v1/call'
  -X 'POST'
  -H 'Authorization: APIAVAIN'
  -H 'X-Phonepoll-Nonce: 123'
  -H 'X-Phonepoll-Signature: ALLEKIRJOITUS'
```

> Korvaa APIAVAIN ja SALAISUUS/ALLEKIRJOITUS omalla API-avaimellasi ja salaisuutesi avulla lasketulla allekirjoituksella.

API-rajapintamme käyttää käyttäjän kaikkien pyyntöjen tunnistamiseen API-avaimia ja viestien vahventamiseen HMAC-SHA256-autentikointia. 

<aside class="warning">
Saat API-avaimen ja autentikoinnin salaisuuden On-Time-yhteyshenkilöltäsi. Pidä niistä hyvää huolta, sillä vastaat täysimääräisesti sinulle myönnetyillä tunnuksilla tehdyistä pyynnöistä myös silloin kun ne ovat joutuneet vääriin käsiin asiakkaan huolimattomuudesta tai tahallisuudesta johtuen.
</aside>

`Authorization: APIAVAIN`

Jokaisessa API-kutsussa tulee olla käyttäjän tunnistava <code>APIAVAIN</code>.

`X-Phonepoll-Nonce`

Jokaisessa API-kutsussa tulee olla tiukasti kasvava nonce-arvo replay-hyökkäysten estämiseksi. Jos teet pyyntöjä enintään sekunnin välein, esimerkiksi Unixin epoch-aikaleima soveltuu nonceksi. Se voi olla myös pyyntöjen järjestysluku. Samaa noncea ei hyväksytä kahdesti eikä edellistä pienempää noncea.

`X-Phonepoll-Signature`

Jokaisen API-kutsun sisällön vahventaa allekirjoitus, joka lasketaan HMAC-SHA256-algoritmillä. Liitä yhdeksi merkkijonoksi pyyntösi JSON-muotoinen POST-data sekä nonce ja aja merkkijono salaisuutesi kanssa HMAC-SHA256-algoritmin läpi.


# Kyselyt

## Soita kysely

```php
<?php

// numeron, josta puhelut lähtevät, täytyy kuulua tilillesi
$fromTelno = "+358942450212";

// kyselyn tunnisteen täytyy olla tilillesi kuuluva kysely
$pollId = 123;

// soitettavat puhelinnumerot, joihin sinulla täytyy olla oikeus soittaa
$numbers = Array();

// määrittele pyynnön data
$payload = new stdClass();
$payload->poll_id = $pollId;
$payload->numbers = $numbers;
$payload->from_telno = $fromTelno;
$requestData = json_encode($payload);

$signature = hash_hmac("sha256", $requestData . $apiNonce, $apiSecret);

$options = array(
    'http' => array(
        'header'  => "Content-Type: application/json\r\n"
                   . "Authorization: APIAVAIN\r\n"
                   . "X-Phonepoll-Nonce: {$apiNonce}\r\n"
                   . "X-Phonepoll-Signature: {$signature}\r\n",
        'method'  => 'POST',
        'content' => $requestData
    )
);
$context  = stream_context_create($options);
$result = file_get_contents("https://puhelinkysely.fi/api/v1/call", false, $context);
if ($result === FALSE) { /* Käsittele virhe */ }

// käsittele vastausdata
var_dump($result);

?>
```

```shell
curl "https://puhelinkysely.fi/api/v1/call"
  -X 'POST'
  -H 'Authorization: APIAVAIN'
  -H 'X-Phonepoll-Nonce: 123'
  -H 'X-Phonepoll-Signature: ALLEKIRJOITUS'
```

> Oheinen pyyntö palauttaa seuraavankaltaisen JSON-muotoisen vastauksen:

```json
{
  "status": "success",
  "error": NULL,
  "poll_id": 123,
  "run_id": 321,
  "from_telno": "+358942450212",
  "calls": [
    {
      "call_id": 444,
      "call_telno": "+358401234567",
      "call_status": "queued"
    },
    {
      "call_id": 445,
      "call_telno": "+358401234568",
      "call_status": "queued"
    }
  ]
}
```

Tällä pyynnöllä voit soittaa ennaltamääritellyn kyselyn yhteen tai useampaan puhelinnumeroon.

### HTTP-pyyntö

`POST https://puhelinkysely.fi/api/v1/call`

### Pyynnön parametrit

Parametri | Tyyppi | Kuvaus
--------- | ------ | ------
poll_id | Integer | Soitettavan kyselyn tunniste
numbers | Array | Soitettavat puhelinnumerot kansainvälisessä E.164-muodossa.
from_telno | String | Numero, josta kyselyt soitetaan. Numeron täytyy olla määritelty tiliisi. Jos parametri puuttuu, käytetään tilisi oletusnumeroa.

### Pyynnön vastaus

Vastaus sisältää soitettavien puheluiden yksilöllisen tunnisteen ja kyselyajon tunnisteen.

Parametri | Tyyppi | Kuvaus
--------- | ------ | ------
status | String | Joko <code>success</code> tai <code>error</code> riippuen pyynnön onnistumisesta.
error | String | Virhetilanteen koodi.
poll_id | Integer | Soitettavan kyselyn tunniste.
run_id | Integer | Kyselyajon tunniste.
from_telno | String | Numero, josta kyselyt soitetaan.
calls | Array | Soitettavat puhelut.
call_id | Integer | Soitettavan puhelun tunniste.
call_telno | String | Soitettava puhelinnumero. Vertaamalla tähän voit tallentaa oikean puhelun tunnisteen itsellesi.
call_status | String | Aina <code>queued</code>, jos kyseisen numeron lisääminen jonoon onnistui. Jos <code>failed</code>, jonoon lisääminen epäonnistui todennäköisimmin virheellisesti muotoillun puhelinnumeron vuoksi.

## Hae kyselyt

```php
<?php

$signature = hash_hmac("sha256", $apiNonce, $apiSecret);

$options = array(
    'http' => array(
        'header'  => "Content-Type: application/json\r\n"
                   . "Authorization: APIAVAIN\r\n"
                   . "X-Phonepoll-Nonce: {$apiNonce}\r\n"
                   . "X-Phonepoll-Signature: {$signature}\r\n",
        'method'  => 'GET'
    )
);
$context = stream_context_create($options);
$result = file_get_contents("https://puhelinkysely.fi/api/v1/polls", false, $context);
if ($result === FALSE) { /* Käsittele virhe */ }

// käsittele vastausdata
var_dump($result);

?>
```

```shell
curl "https://puhelinkysely.fi/api/v1/polls"
  -X 'GET'
  -H 'Authorization: APIAVAIN'
  -H 'X-Phonepoll-Nonce: 123'
  -H 'X-Phonepoll-Signature: ALLEKIRJOITUS'
```

> Oheinen pyyntö palauttaa seuraavankaltaisen JSON-muotoisen vastauksen:

```json
[
  {
    "poll_id": 123,
    "poll_title": "Esimerkkikysely",
    "questions": [
      {
        "question": "Oletko samaa mieltä?",
        "choices": [
          {
            "choiceText": "Jos kyllä, paina yksi.",
            "choiceSelection": "Kyllä"
          },
          {
            "choiceText": "Jos et, paina kaksi.",
            "choiceSelection": "Ei"
          }
        ]
      }
    ]
  }
]      
```

Tällä pyynnöllä voit hakea kaikki tilillesi määritellyt kyselyt. Pyynnön palauttamaa kyselyn tunnistetta voi käyttää parametrina [kyselyiden soitossa](#soita-kysely).

### HTTP-pyyntö

`GET https://puhelinkysely.fi/api/v1/polls`

### Pyynnön parametrit

Pyyntö ei odota parametreja, vaan palauttaa kaikki tilillesi määritellyt kyselyt.

### Pyynnön vastaus

Vastaus sisältää kaikki tilillesi määritellyt kyselyt.

Parametri | Tyyppi | Kuvaus
--------- | ------ | ------
poll_id | Integer | Kyselyn tunniste.
poll_title | String | Kyselyn otsikko.
questions | Array | Kyselyn kysymykset.
question | String | Kysymysteksti.
choices | Array | Kysymyksen vastausvaihtoehdot.
choiceText | String | Vastausvaihtoehdon teksti.
choiceSelection | String | Vastausvaihtoehdon raportille tuleva otsake.
