## Mongo 导出数据
bin/mongoexport --host=10.28.xx.xx:27017 -d push -c device -q '{"_id":{$gt:1004146701}}'  -o result.dat

数据量比较大时直接import，比restore要快很多

## Mongo 导入数据
bin/mongoimport --host=172.16.6.226 --db push --collection device --file result.json

