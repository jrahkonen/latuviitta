## Latuviitan tuhat GDAL-ohjetta

(työn alla, valmiina 1,5 %)

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


