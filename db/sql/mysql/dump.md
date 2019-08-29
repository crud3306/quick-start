

批量导出数据
mysqldump -u USER -p test $(mysql -u USER -p -D test -Bse "show tables like 'wiki_%'")

