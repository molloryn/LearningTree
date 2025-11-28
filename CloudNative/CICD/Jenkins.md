# Jenkins


## 问题排查

### Pipeline


#### Pipeline中 cat app.log 无输出
缓冲区没来得及刷新日志，需要`sleep 5` （2秒都不够）