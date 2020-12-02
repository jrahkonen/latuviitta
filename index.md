## Latuviitan GDAL-ohjeita

GDAL-ohjelmilla voi tehdä hämmästyttäviä asioita ilman ohjelmointitaitojakin.


### Tuhat ensimmäistä GDAL-tehtävää

|Numero| Tehtävä|      
|---|---|
|1| Lisää uusi sarake shapefileen|
||ogrinfo -sql "alter table states add wkt_geom text" states.shp
|2| Ota polygonin keskipiste, vaihda sen koordinaattijärjestelmä, ja tallenna tulos WKT-tekstinä haluttuun shapefilen kenttään
||ogrinfo -dialect sqlite -sql "update states set wkt_geom=ST_AsText(ST_Transform(ST_Centroid(geometry),3857))" states.shp
 
