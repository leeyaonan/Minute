一、适用于`field`为一个唯一索引的情况：
```mysql
INSERT INTO `table`(`field`) VALUES (content1) ON DUPLICATE KEY UPDATE `field` = content2;
```

