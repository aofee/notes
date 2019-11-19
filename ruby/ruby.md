### ruby 写sql块
```ruby
sql = <<-SQL.strip_heredoc
SQL

query = ActiveRecord::Base.connection.execute(sql)
if query.values.first.present?
    query_result = query[0]
end
```

### rails data-migrate

https://github.com/ilyakatz/data-migrate

```shell
# 生成data_migration
rails g data_migration add_this_to_that
# 生成migration
rails g migration add_column_to_table
# 按照生成时间交替执行migrate和生成data_migration
rake db:migrate:with_data
```



### pg-postgis postgres geo插件 地理位置插件
```shell
docker pull kartoza/postgis:10.0-2.4
docker run --name pg_gis -d -v /home/t04872/pg/data/pg_gis/:/var/lib/postgresql/ -e POSTGRES_DBNAME=pg_gis -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=postgres -p 54322:5432 kartoza/postgis:10.0-2.4
```
往postgis中存储坐标系
```sql
-- 首先建立一个常规的表格存储有关城市（cities）的信息。这个表格有两栏，一个是 ID 编号，一个是城市名：
CREATE TABLE cities ( id int4, name varchar(50) );
-- 现在添加一个空间栏用于存储城市的位置。习惯上这个栏目叫做 the_geom 。它记录了数据为什么类型（点、线、面）、有几维（这里是二维）以及空间坐标系统。此处使用 EPSG:4326 坐标系统：
SELECT AddGeometryColumn ('cities', 'the_geom', 4479, 'POINT', 2);


-- 为添加记录，需要使用 SQL 命令。对于空间栏，使用 PostGIS 的 ST_GeomFromText 可以将文本转化为坐标与参考系号的记录
INSERT INTO cities (id, the_geom, name) VALUES (1,st_geometryfromtext('POINT(121.455231 31.249791)',4479),'上海市静安区秣陵路303号1');
INSERT INTO cities (id, the_geom, name) VALUES (2,ST_GeomFromText('POINT(31.177853 121.422272)',4326),'上海市徐汇区宜山路600号');
INSERT INTO cities (id, the_geom, name) VALUES (3,ST_GeomFromText('POINT(31.196651 121.346397)',4326),'上海市长宁区迎宾一路');
INSERT INTO cities (id, the_geom, name) VALUES (4,ST_GeomFromText('POINT(31.187326 121.629124)',4326),'上海市浦东新区张江路1618号');
INSERT INTO cities (id, the_geom, name) VALUES (5,ST_GeomFromText('POINT(31.215444 121.551704)',4326),'上海市浦东新区锦绣路1001号');
INSERT INTO cities (id, the_geom, name) VALUES (6,ST_GeomFromText('POINT(31.239811 121.500549)',4326),'上海市浦东新区陆家嘴环路1386号');


INSERT INTO cities (id, the_geom, name) VALUES (6,ST_GeomFromText('POINT(31.222784 121.504540)',4326),'上海市黄浦区复兴东路1号');
INSERT INTO cities (id, the_geom, name) VALUES (7,ST_GeographyFromText('SRID=4326;POINT(31.239811 121.500549)',4326),'上海市浦东新区陆家嘴环路1386号');

-- 这里的坐标是无法阅读的 16 进制格式。要以 WKT 文本显示，使用 ST_AsText(the_geom) 或 ST_AsEwkt(the_geom) 函数。也可以使用 ST_X(the_geom) 和 ST_Y(the_geom) 显示一个维度的坐标：
SELECT id, ST_AsText(the_geom), ST_AsEwkt(the_geom), ST_X(the_geom), ST_Y(the_geom) FROM cities;

-- 查询到该点的距离
SELECT
    name,ST_X(st_transform(the_geom,4479)),
    st_distance(the_geom, st_transform(ST_GeometryFromText('POINT(121.523895 31.222197)',4326),4479)
        ) AS distance FROM organizations
ORDER BY
    distance
LIMIT 10



SELECT
    name,ST_AsText(the_geom),ST_distance(the_geom::geography,ST_GeomFromText('POINT(31.219701 121.505699)',4326)) AS distance 
FROM cities
ORDER BY
    distance
LIMIT 5

```

### postgres 导入导出数据

#### 导出
```shell
-s  选项用来只导出表结构，而不会导出表中的数据
-t   选项用来指定要导出的数据库表

格式：docker exec -it 容器名 pg_dump -U 用户名 -s -t table_name db_name > sql文件保存位置
docker exec -it postgres pg_dump -U postgres uaa_development > ~/uaa_development.sql

如果是远程连接，添加 -h -p参数
docker exec -it group-postgres pg_dump  -h host  -p port -U leaniot -s -t quake_info gis > ./t.sql
```

#### 导入

1. docker cp 命令，把sql文件copy到容器内部根目录

2. docker exec 执行 导入数据库命令

```shell
docker cp your.sql group-postgres:/
docker exec group-postgres sh -c 'exec psql -U username -d dbname < your.sql'
```

如果提示 Peer authentication failed for user
我们需要进入容器里面 切换到postgres用户再次执行

```shell
docker exec -it postgres /bin/bash
su postgres
psql -d uaa_development < uaa_development.sql
```




geo 

name,address,response,lng,lat,reliability


```json

```

### bundle install 过程中的错误

+ An error occurred while installing pg (0.17.1), and Bundler cannot continue
```shell
sudo apt-get install libpq-dev 
```
+ An error occurred while installing rmagick (4.0.0), and Bundler cannot continue.
```shell
https://stackoverflow.com/questions/21901657/an-error-occurred-while-installing-rmagick-2-13-2-and-bundler-cannot-continue/21901885
sudo apt-get install libmagickwand-dev
```