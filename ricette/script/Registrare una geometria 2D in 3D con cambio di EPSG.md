# Come registrare una geometria 2D in 3D con cambio di EPSG 

* autore: _[Giuseppe Guarino](https://www.linkedin.com/in/guarino-giuseppe/)_
* issue: [#54](https://github.com/opendatasicilia/tansignari/issues/54) fornitore ricetta *[Totò Fiandaca](https://twitter.com/totofiandaca?lang=it)*
* ingredienti: [SpatiaLite](https://www.gaia-gis.it/fossil/libspatialite/index)

---

<!-- TOC -->

- [Come registrare una geometria 2D in 3D con cambio di EPSG](#come-registrare-una-geometria-2d-in-3d-con-cambio-di-epsg)
  - [Introduzione](#introduzione)
  - [Dataset](#dataset)
  - [Step 1 - Avviare SpatiaLite](#step-1---avviare-spatialite)
  - [Step 2 - Inserire i nostri dati](#step-2---inserire-i-nostri-dati)
  - [Step 3 - Effettuare la query](#step-3---effettuare-la-query)

<!-- /TOC -->

---

Come crare una geo-tabella 3D con EPSG 6708 a partire da uno shapefile 2D importato in SpatiaLite con campi X, Y e Z.

---

## Introduzione

Dopo aver effettuato un rilievo topografico con un GPS differenziale, spesso abbiamo bisogno di convertire le nostre coordinate geografiche espresse in Latitudine e Longitudine, 
in coordintate cartografiche piane.

In questa ricetta effettueremo una conversione con [SpatiaLite](https://www.gaia-gis.it/fossil/libspatialite/index).

## Dataset
* [SpatiaLite-gui](http://www.gaia-gis.it/gaia-sins/windows-bin-NEXTGEN-amd64/)
Disponibile per versioni: Windows, Linux e Mac
* [punti_prova (shp)](https://mega.nz/#!cZ4zRIxA!bWiURwS97ssIP6hR1wc1iQwmP2I2TqAiaWNBaAF-Vvk)

## Step 1 - Avviare SpatiaLite

SpatiaLite non ha bisogno di nessuna installazione, quindi basta scaricare l'eseguibile e avviarlo.
Dopo averlo avviato bisogna **creare un nuovo database** dal Menu a tendina cliccando su *Creating a New (empty) SQLiteDB*.

## Step 2 - Inserire i nostri dati

Dopo aver creato il nostro database dobbiamo caricare i nostri dati (CSV o SHP), dall'apposito tasto di importazione, oppure da *Menu* - *Advanced* - *Load Shapefile* (o csv/txt).

## Step 3 - Effettuare la query

Per effettuare la trasformazione basta copiare e incollare la query qui sotto, avendo cura di sostituire:
* `nome tabella` (con il nome di una nuova tabella)
* `myTable` (con il nome della tabella/shapefile caricato)

```sql
CREATE TABLE "nome_tabella" AS 
SELECT nome, x, y, z, ST_Transform(geom, SRID) AS geom
FROM "myTable";
```

la query crea una geo-tabella (**nome_tabella**) seguendo la geometria dello shapefile importato (**myTable**) e trasformando solo EPSG, quindi se la geometria era XY rimarrà XY.

La seguente query effettua la trasformazione:

```sql
CREATE TABLE "nome_tabella_new" AS 
SELECT nome, x, y, z, makepointz(ST_x(geom), ST_y(geom), cast(z as double),6708) as geom
FROM "nome_tabella";
```

a partire dalla precedente geo-tabella (**nome_tabella**) con EPSG trasformato, crea una nuova geo-tabella aggiungendo la terza coordinata alla geometria XY trasformandola in XYZ; infine occorre registrare la geometria tramite questa query:

```sql
SELECT RecoverGeometryColumn('nome_tabella_new', 'geom',6708, 'POINT', 'XYZ');
```
