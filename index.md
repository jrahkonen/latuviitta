## Latuviitan tuhat GDAL-ohjetta

(työn alla, valmiina 4,0 %)

GDAL-ohjelmilla voi tehdä hämmästyttäviä asioita ilman ohjelmointitaitojakin.


### GDAL-ohjelmien käyttöesimerkkejä

|Numero| Tehtävä|
|---|---|
|1| Lisää uusi sarake shapefileen.
||ogrinfo -sql "alter table states add wkt_geom text" states.shp
|2| Ota polygonin keskipiste, vaihda sen koordinaattijärjestelmä, ja tallenna tulos WKT-tekstinä haluttuun shapefilen kenttään.
||ogrinfo -dialect sqlite -sql "update states set wkt_geom=ST_AsText(ST_Transform(ST_Centroid(geometry),3857))" states.shp
|3| Tiivistä GeoPackage vapauttamalla poistettujen rivien vaatima levytila.
||ogrinfo -sql "VACUUM" oma_geopackage.gpkg
|4| Selvitä, mitä GDAL-versiota käytät.
||gdalinfo --version
|5| Selvitä, mitkä SQLite- ja SpatiaLite-versiot ovat käytössä - joku olemassa oleva tietolähde on osoitettava, se voi olla mikä tahansa.
||ogrinfo -dialect SQLite -sql "select sqlite_version(), spatialite_version()" foo.shp
|6| Selvitä, mitä ajureita käytössä oleva GDAL tukee - ogrinfo vektoriformaateille, gdalinfo rasteriformaateille.
||gdalinfo --formats
|7| Selvitä tietyn ajurin ominaisuudet - vastaus XML-muodossa.
||ogrinfo --format gml
|8| Peruskomento, jolla saa aikaan QGIS:ssä, GeoServerissä ym. hyvin toimivan GeoTIFF:in. Häviötön pakkaus.
||gdal_translate -of GTiff -co tiled=yes -co compress=deflate input.tif output.tif
|9| Peruskomento ilma- ja satelliittikuvien muuntamiseksi. JPEG-pakkaus - pieni tiedostokoko.
||gdal_translate -of Gtiff -co tiled=yes -co compress=jpeg --config photometric=ycbcr input.tif output.tif
|10| Ilmakuville hyvä komento overview-tasojen lisäämiseksi - tehokas pakkaus myös niille.
||gdaladdo -r average --config COMPRESS_OVERVIEW JPEG --config PHOTOMETRIC_OVERVIEW YCBCR output.tif
|11| PostGIS:in yhteyden syntaksia ei vaan millään tahdo muistaa ulkoa.
||ogrinfo PG:"host=localhost port=5432 dbname=oma_postgis user=käyttäjä password=salasana"
|12| Lisätään PostGIS-tauluun toinen geometriakenttä.
||ogrinfo PG:"host=localhost port=5432 dbname=oma_postgis user=käyttäjä password=salasana" -sql "alter table testitaulu add column geom2 geometry"
|13| Päivitetään toinen geometriakenttä ja kirjoitetaan siihen toiseen koordinaattijärjestelmään muunnetut geometriat.
||ogrinfo PG:"host=localhost port=5432 dbname=oma_postgis user=käyttäjä password=salasana" -sql "update testitaulu set geom2=ST_Transform(geom,3857)"
|14| FeatureID on normaalisti piilotettu. Näin sen saa tavalliseksi attribuutiksi OGRSQL-murteella.
||ogr2ogr -f CSV -sql "select FID, * from testi_shape" output.csv testi_shape.shp
|15| FID on helpointa etsiä OGRSQL-murteella, jolle sen nimi on aina "FID". Tietokantojen omilla SQL-murteilla sekä GDAL:in SQLite-murteella täytyy käyttää FID-kentän oikeaa nimeä, mutta aina sitä ei ole olemassa. Shapefilen FID löytyy SQLite-murteelle näin.
||ogr2ogr -f CSV -dialect SQLite -sql "select rowid as feature_id, * from testi_shape" output.csv testi_shape.shp
|16| Lue tietoja suoraan zip-arkistosta, jossa on GDAL:in tunnistamia paikkatietoaineistoja
||ogrinfo /vsizip/zipattu_geopackage.zip
|17| Lue tietoja zip-arkistosta, joka on netissä http-yhteyden takana
||ogrinfo /vsizip/vsicurl/"http://latuviitta.org/downloads/Pyhajarvi_001.zip"
|18| Tee GeoTIFF-kuvasta kopio, josta on riisuttu pois kaikki GeoTIFF-tagit
||gdal_translate -of GTiff -co profile=baseline --config GDAL_PAM_ENABLED NO input.tif puhdas_tif.tif
|19| Shapefile sisältää sekä polygoneja että multipolygoneja. Tallenna ne sellaisenaan formaattiin, joka voi asettaa geometriatyypille rajoitteita (esim. PostGIS, GeoPackage)
||ogr2ogr -f GPKG oma_geopackage.gpkg polygon.shp -nlt GEOMETRY
|20| Sama tilanne kuin edellä, mutta muunnetaan kaikki polygonit multipolygon-muotoon
||ogr2ogr -f GPKG oma_geopackage.gpkg polygon.shp -nlt PROMOTE_TO_MULTI
|21| Sama tilanne edelleen, mutta hajoitetaan multipolygonit (kohteiden määrä kasvaa) ja tallennetaan kaikki polygoneina
||ogr2ogr -f GPKG oma_geopackage.gpkg polygon.shp -explodecollections
|22| Kahden lon-lat -pisteen välinen etäisyys metreinä ellipsoidin pintaa pitkin mitattuna
||ogrinfo -dialect SQLite -sql "SELECT ST_Distance(ST_GeomFromText('POINT (130.4572 33.0115)'),ST_GeomFromText('POINT (130.4572 33.0063)'),1)" foo.jml
|23| Tee satunnainen piste, jonka x ja y ovat kokonaislukuja väliltä 0-100
||ogrinfo -dialect sqlite -sql "select st_geomfromtext('POINT ('\|\|abs(random() % 100) \|\|' '\|\|abs(random() % 100)\|\|')')" foo.jml
|24| Tee uusi GeoPackage ja siihen taulu, jossa on 10000 pistettä annetun polygonin sisällä (vaatii, että PostGIS on saatavilla)
||ogr2ogr -f gpkg random_points.gpkg PG:"host=localhost port=5432 dbname=database user=käyttäjä password=salasana" -sql "SELECT ST_GeneratePoints(geom, 10000, 1996) FROM (SELECT ST_Buffer(ST_GeomFromText('LINESTRING(50 50,150 150,150 50)'),10, 'endcap=round join=round') AS geom) AS s" -explodecollections -nln points
|25| Muunnelma: peitä polygonin alue neliöillä
||ogrinfo -dialect SQLite -sql "SELECT ST_SquareGrid(geom, 10) FROM (SELECT ST_Buffer(ST_GeomFromText('LINESTRING(50 50,150 150,150 50)'),10) AS geom) AS s" foo.jml
|26| Muunnelma: peitä polygonin alue kolmioilla
||ogrinfo -dialect SQLite -sql "SELECT ST_TriangularGrid(geom, 10) FROM (SELECT ST_Buffer(ST_GeomFromText('LINESTRING(50 50,150 150,150 50)'),10) AS geom) AS s" foo.jml
|27| Muunnelma: peitä polygonin alue kuusikulmiolla
||ogrinfo -dialect SQLite -sql "SELECT ST_HexagonalGrid(geom, 10) FROM (SELECT ST_Buffer(ST_GeomFromText('LINESTRING(50 50,150 150,150 50)'),10) AS geom) AS s" foo.jml
|28| Selvitä, onko TIFF-kuvan tilastotiedot kuvan tilastotiedot (MIN, MAX, MEAN, STDDEV) jo laskettu
||gdalinfo kuva.tif|
||Jos tilastotiedot näkyvät, niin ne on laskettu. Jos on olemassa .aux.xml-tiedosto, niin ne on mahdollisesti tallennettu siihen. Jos aux.xml-tiedostoa ei ole, niin tilastotiedot on tallennettu TIFF-tiedoston tageihin.
|29| Poista vanhat, mahdollisesti väärät tilastotiedot ennen niiden päivittämistä. Komento joko tuhoaa .aux.xml-tiedoston tai päivittää TIFF-tiedoston tagit. (Huom. Python-skripti).
||gdal_edit -unsetstats kuva.tif
|30| Laske ja näytä kuvan tilastotiedot sekä tallenna ne ulkoiseen aux.xml -tiedostoon 
||gdalinfo -stats kuva.tif
|31| Muunnelma: Laske ja näytä nopeat, mutta epätarkemmat tilastotiedot otoksen perusteella
||gdalinfo -approx_stats kuva.tif
|32| Muunnelma: Näytä tilastotiedot, mutta älä tallenna niitä
||gdalinfo -stats kuva.tif --config GDAL_PAM_ENABLED NO
|33| Muunnelma: Laske tilastotiedot ja tallenna ne TIFF-tiedoston sisään niille varattuihin tageihin (Huom. Python-skripti)
||gdal_edit -stats kuva.tif
|34| Luo uusi yhdellä värillä täytetty TIFF-tiedosto, jonka ulottuvuudet ja georeferointi kopioidaan olemassa olevasta kuvasta.
||gdal_create -burn 255 0 0 -if P4433H.tif punainen_ruutu.tif
|35| Sama, mutta ilman mallikuvaa
||gdal_create -of GTiff -outsize 12000 12000 -bands 3 -burn 0 0 255 -ot Byte -a_srs epsg:3067 -a_ullr 494000 7014000 500000 7008000 sininen_ruutu.tif
|36| Polta punaisen kuvan keskelle vihreä kierretty neliö, joka on luotu SQL:n avulla lennossa
||gdal_rasterize -b 1 -b 2 -b 3 -burn 0 -burn 255 -burn 0 -dialect SQLite -sql "select ST_GeomFromText('POLYGON (( 494000 7008500, 494500 7014000, 500000 7013500, 499500 7008000, 494000 7008500 ))')" foo.jml punainen_ruutu.tif
|37| Polta siniseen kuvaan mustaa annetun kierretyn neliön ulkopuolelle. Tulos simuloi uudelleen projisoitua rasterikuvaa, jonka nurkissa on nodata-kiilat.
||gdal_rasterize -b 1 -b 2 -b 3 -burn 0 -burn 0 -burn 0 -i -dialect SQLite -sql "select ST_GeomFromText('POLYGON (( 494000 7008500, 494500 7014000, 500000 7013500, 499500 7008000, 494000 7008500 ))')" area.jml sininen_ruutu.tif
|38| Hae Geofabrik:in latauspalvelusta uusin Suomen OpenStreetMap-aineisto ja tallenna se GeoPackage-muotoon.
||ogr2ogr -f GPKG -t_srs EPSG:3067 osm_finland_uusi.gpkg /vsicurl_streaming/http://download.geofabrik.de/europe/finland-latest.osm.pbf
|39| Muunna JPEG-pakattu RGB-kuva (YCBCr) harmaasävykuvaksi "ITU-R Recommendation BT.601" suosituksen mukaisella kaavalla
| |gdal_calc.py -R input.tif --R_band=1 -G input.tif --G_band=2 -B input.tif --B_band=3 --outfile=result.tif --calc="R\*0.2989+G\*0.5870+B\*0.1140"
|40| Hae http-palvelimelta kaksi TIFF-kuvaa, vertaile niitä, ja tee tulosrasteri niistä pikseleistä, joiden arvo molemmissa kuvissa on "17".
||gdal_calc.py -A /vsicurl/https://vm0160.kaj.pouta.csc.fi/syke/corine/2012/corine_2012_100000_6700000.tif -B /vsicurl/https://vm0160.kaj.pouta.csc.fi/syke/corine/2006/corine_2006_100000_6700000.tif --outfile=pellot5.tif --calc="logical_and(A==17,B==17)" --type=Byte
|41| Säästä vähän tilaa poistamalla Z-koordinaatit, jos niille ei ole käyttöä.
||ogr2ogr -f "ESRI Shapefile" XY.shp XYZ.shp -dim XY
|42| Yleistä viiva- ja aluegeometriat Douglas-Peucker-menetelmällä 10 mittayksikön toleranssilla
||ogr2ogr -f JML -simplify 10 yleistetty.jml yleistämätön.jml
|43| Tihennä geometrioita tarvittaessa niin, että taitepisteiden väli on korkeintaan 50 yksikköä. Usein hyödyllinen, jos geometriolle tehdään joko samalla kertaa tai myöhemmin projektionmuunnos.
||ogr2ogr -f JML -segmentize 50 tihennnetty.jml harva.jml
|44| Muunna aineisto niin, että topologiavirheitä sisältävät geometria samalla korjataan.
||ogr2ogr -f JML -makevalid korjattu.jml viallinen.jml
|45| Testaa jättimäisen aineiston muunnosta ensin sadalla kohteella
||ogr2ogr -f JML -limit 100 pikkuotos.jml jättigeopackage.gpkg jättitaulu
