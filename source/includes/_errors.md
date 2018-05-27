# Virheet

## HTTP-virhekoodit

API-rajapinta palauttaa seuraavia HTTP-virhekoodeja.

Virhekoodi | Merkitys
---------- | --------
400 | Bad Request -- Pyyntösi on virheellinen tai virheellisesti muotoiltu.
401 | Unauthorized -- API-avaimesi tai allekirjoituksesi on virheellinen.
403 | Forbidden -- Käyttöoikeustasosi ei ole riittävä.
404 | Not Found -- Pyydettyä osoitetta tai tietoa ei löytynyt.
405 | Method Not Allowed -- Virheellinen metodi.
406 | Not Acceptable -- Virheellinen muoto. Pyydettävän muodon tulee olla aina JSON.
410 | Gone -- Pyydetty tieto on poistettu pysyvästi.
418 | I'm a teapot. -- Olen teekannu.
429 | Too Many Requests -- Teet pyyntöjä liian usein-
500 | Internal Server Error -- Palvelimessa on häiriö tai virhetilanne. Odota ja kokeile myöhemmin uudelleen.
503 | Service Unavailable -- Olemme väliaikaisesti huollossa. Odota ja kokeile myöhemmin uudelleen.

## Pyyntöjen virhekoodit

Jos pyyntö on muodollisesti oikein, mutta sen toteutus epäonnistuu, vastauksen <code>error</code>-kenttä sisältää jonkin seuraavista virhekoodeista.

Virhekoodi | Merkitys
---------- | --------
INVALID_TELNO | Virheellinen tai puuttuva puhelinnumero.
INVALID_POLL_ID | Virheellinen tai puuttuva kyselyn tunniste.
INVALID_FROM_TELNO | Virheellinen lähtevä puhelinnumero.
INVALID_API_KEY | Virheellinen tai tuntematon API-avain.
INVALID_API_SIGNATURE | Virheellinen allekirjoitus.
INVALID_NONCE | Virheellinen, liian vanha tai uudelleenkäytetty nonce.