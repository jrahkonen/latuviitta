## Latuviitan tuhat GDAL-ohjetta

(työn alla, valmiina 2,7 %)

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
