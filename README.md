## What is this?

A nginx module to serve map tiles directly from mbtiles container files. This is a fast, low-resource way to serve many mbtiles files (tested with 11,000 mbtiles files on a 2 GB/1 CPU VPS), especially if the contents of those file changes regularly: nginx performs the open/fetch calls on each request, so adding new mbtiles files or updating their contents requires no action on your part for those to be served immediately.

## Nginx configuration example

* mbtiles_file - points to the mbtiles file

You'll need to add `load_module modules/ngx_http_mbtiles_module.so;` to the root of your nginx.conf file.

<pre>
  location ~ ^/(.*?)/(.*?)/(.*?)/(.*?)$ {
      root /var/www/tiles/tilesets;
      mbtiles_file "$document_root/$1.mbtiles";
      mbtiles_zoom "$2";
      mbtiles_column "$3";
      mbtiles_row "$4";
  }

</pre>

## Installation

### Prerequisites

You need to have a `sqlite3-dev` package installed. On Ubuntu or Debian you can install it using:

```sh
apt-get install libsqlite3-dev
```

## Limitations

At the moment this expects that the tile content fetched from the mbtiles file is gzip compressed. 

There may be some security issues around the regex matching ($1/$2/$3/$4), but 1) casting tile coordinates to `short`s before fetching from the mbtiles should reduce some of these concerns, 2) you should probably not have sensitive data mixed in with your tilesets (ie serve tiles from a dedicated tile directory and set your document root to be the base of this directory).

## Changes

11/2024: Changed `tile_zoom` to `char` (since zoom levels never exceed 127) and `tile_row`, `tile_column`, `tms_tile_row` to `int` (to support coordinates higher than 32767).

08/2020: Various cleanups: restricted the $2/$3/$4 references to be numeric and implemented ZXY->TMS translation so that this module would work with libraries like Mapbox GL JS that request tiles in ZXY notation. Explicitly finalize the SQLite statement to close a memory leak. Changed missing tile HTTP response code to 204 No Content instead of 404 Not Found.

Much of the work on this was originally done by @pace (~2017). 
