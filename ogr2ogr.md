Collection of ogr2ogr commands

Nro|Tehtävä
---|---
|| Tehdään PostGIS-tietokantaan uusi taulu, jolle annetaan haluttu nimi
|| `ogr2ogr -f PostgreSQL PG:"host=localhost port=5432 dbname=oma_postgis user=käyttäjä password=salasana" aineisto.jml -nln testitaulu`
|| Tehtään PostGIS-kantaan uusi taulu jonka nimi on jo käytössä. Koska ensin tapahtuu "DROP TABLE", niin skeema voi muuttua.
|| `ogr2ogr -f PostgreSQL -overwrite PG:"host=localhost port=5432 dbname=oma_postgis user=käyttäjä password=salasana" aineisto.jml -nln testitaulu`
|| Tyhjennetään olemassa oleva PostGIS-taulu ja kirjoitetaan siihen uusi sisältö.  Koska ensin tapahtuu "TRUNCATE TABLE", niin skeeman on oltava sama.
|| `ogr2ogr -f PostgreSQL -append –-config OGR_TRUNCATE YES PG:"host=localhost port=5432 dbname=oma_postgis user=käyttäjä password=salasana" aineisto.jml -nln testitaulu`
|| Ei poisteta eikä tyhjennetä olemassa olevaa PostGIS-taulua, vaan lisätään sen jatkeeksi uutta sisältöä. Skeeman on oltava sama.
|| `ogr2ogr -f PostgreSQL -append PG:"host=localhost port=5432 dbname=oma_postgis user=käyttäjä password=salasana" aineisto.jml -nln testitaulu`
|| Lisätään olemassa olevan taulun jatkeeksi uutta sisältöä. Jos skeemat eivät ole samat, niin PostGIS-tauluun lisätään uusissa tiedoissa esiintyvät uudet attribuutit.
|| `ogr2ogr -f PostgreSQL -append -addfields PG:"host=localhost port=5432 dbname=oma_postgis user=käyttäjä password=salasana" aineisto.jml -nln testitaulu`
