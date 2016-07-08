# hdp-hive-spatial
Hive Spatial Queries with ESRI Libraries

## Steps
1. Clone this repo locally:

`git clone https://github.com/cstanca1/hdp-hive-spatial.git' 
'

2. Extract the 3 jars files from esri.zip and copy them to hdfs. Grant ownership on these jar files to hive:hdfs.

3. Connect to your hive database using your preferred CLI and execute the content of spatial-geometry-demo-ddl-dml.txt to create tables and populate with sample data.

4. Before executing any query, add ESRI jar libraries to your session execution path, for the session or globally.


`
add jar hdfs://YourHDFSClientNode:8020/esri/esri-geometry-api.jar;

add jar hdfs://YourHDFSClientNode:8020/esri/spatial-sdk-hive-1.1.1-SNAPSHOT.jar;

add jar hdfs://YourHDFSClientNode:8020/esri/spatial-sdk-json-1.1.1-SNAPSHOT.jar;
'

5. Define temporary functions for better SQL-like experience.

`
create temporary function st_geomfromtext as 'com.esri.hadoop.hive.ST_GeomFromText';

create temporary function st_geometrytype as 'com.esri.hadoop.hive.ST_GeometryType';

create temporary function st_point as 'com.esri.hadoop.hive.ST_Point';

create temporary function st_asjson as 'com.esri.hadoop.hive.ST_AsJson';

create temporary function st_asbinary as 'com.esri.hadoop.hive.ST_AsBinary';

create temporary function st_astext as 'com.esri.hadoop.hive.ST_AsText';

create temporary function st_intersects as 'com.esri.hadoop.hive.ST_Intersects';

create temporary function st_x as 'com.esri.hadoop.hive.ST_X';

create temporary function st_y as 'com.esri.hadoop.hive.ST_Y';

create temporary function st_srid as 'com.esri.hadoop.hive.ST_SRID';

create temporary function st_linestring as 'com.esri.hadoop.hive.ST_LineString';

create temporary function st_pointn as 'com.esri.hadoop.hive.ST_PointN';

create temporary function st_startpoint as 'com.esri.hadoop.hive.ST_StartPoint';

create temporary function st_endpoint as 'com.esri.hadoop.hive.ST_EndPoint';

create temporary function st_numpoints as 'com.esri.hadoop.hive.ST_NumPoints';
`

6. Execute the various spatial queries included in spatial-geometry-demo-queries.txt file also listed below:

Counts by geometry type -- assumes that other than ST_POINT values are possible:

`
select st_geometrytype(st_geomfromtext(shape)), count(shape)
from demo_shape_point
group by st_geometrytype(st_geomfromtext(shape));
`

Counts by geometry type -- assumes that other than ST_LINESTRING values are possible:

`
select st_geometrytype(st_geomfromtext(shape)), count(shape)
from demo_shape_linestring
group by st_geometrytype(st_geomfromtext(shape));
`

Counts by geometry type -- assumes that other than ST_POLYGON values are possible:

`
select st_geometrytype(st_geomfromtext(shape)), count(shape)
from demo_shape_polygon
group by st_geometrytype(st_geomfromtext(shape));
`

X and Y coordinates of the point:

`
select st_x(st_point(shape)) AS X, st_y(st_point(shape)) AS Y 
from demo_shape_point 
where st_geometrytype(st_geomfromtext(shape)) = "ST_POINT"
limit 1;
`

Extract geometry from text shape:

`
select st_geomfromtext(shape) 
from demo_shape_point 
where st_geometrytype(st_geomfromtext(shape)) = "ST_POINT"
limit 1;
`

Geometry type:

`
select st_geometrytype(st_geomfromtext(shape)) 
from demo_shape_point 
where st_geometrytype(st_geomfromtext(shape)) = "ST_POINT"
limit 1;
`

Point geometry as a binary - implicitly:

`
select st_point(shape) 
from demo_shape_point 
where st_geometrytype(st_geomfromtext(shape)) = "ST_POINT" 
limit 1;
`

Point geometry as a binary - explicitly:

`
select st_asbinary(st_geomfromtext(shape)) 
from demo_shape_point 
where st_geometrytype(st_geomfromtext(shape)) = "ST_POINT"
limit 1;
`

Point geometry as Json:
`
select st_asjson(st_geomfromtext(shape)) 
from demo_shape_point 
where st_geometrytype(st_geomfromtext(shape)) = "ST_POINT"
limit 1;
`

Point geometry as a text:

`
select st_astext(st_point(shape)) 
from demo_shape_point 
where st_geometrytype(st_geomfromtext(shape)) = "ST_POINT"
limit 1;
`

SRID for a point:

`
select distinct st_srid(st_point(shape))
from demo_shape_point
where st_geometrytype(st_geomfromtext(shape)) = "ST_POINT"
limit 1;
`

Line as text:

`
select st_astext(st_linestring(shape))
from demo_shape_linestring
where st_geometrytype(st_geomfromtext(shape)) = "ST_LINESTRING"
limit 1;
`

n point of a line:

`
select st_astext(st_point(st_astext(st_pointn(st_linestring(shape), 2))))
from demo_shape_linestring
where st_geometrytype(st_geomfromtext(shape)) = "ST_LINESTRING"
limit 1;
`

Start and end points of a line:

`
select st_astext(st_startpoint(st_linestring(shape))) AS StartPoint, 
       st_astext(st_endpoint(st_linestring(shape))) AS EndPoint
from demo_shape_linestring
where st_geometrytype(st_geomfromtext(shape)) = "ST_LINESTRING"
limit 1;
`

Number of points in a polygon:

`
select shape, st_numpoints(st_geomfromtext(shape)) as NumPoints
from demo_shape_polygon
where st_geometrytype(st_geomfromtext(shape)) = "ST_POLYGON"
limit 1;
`

Lines intersection - usually you would have two tables - this is just an example with a table with one row:

`
select st_intersects(a.s1, b.s2)
from 
   (select st_point(shape) AS  `s1` 
      from demo_shape_point limit 1) a join
   (select st_point(shape) AS  `s2` 
      from demo_shape_point limit 1) b
limit 1;
`

Lines intersection - usually you would have two tables - this is just an example with a table with one row:

`
select st_intersects(a.s1, b.s2)
from 
   (select st_linestring(0,0, 1,1) AS  `s1` 
      from demo_shape_linestring limit 1) a join
   (select st_linestring(1,1, 0,0) AS  `s2` 
      from demo_shape_linestring limit 1) b
limit 1;
`

## Final Note

For more UDFs go to: https://github.com/Esri/spatial-framework-for-hadoop/wiki/UDF-Documentation.