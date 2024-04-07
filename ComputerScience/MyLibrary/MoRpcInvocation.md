
```json
 "Services": {
   "FlightService.API": {
     "AppId": "service-flight-api"
   },
   "FlightService.ActorHost": {
     "AppId": "service-flight-actorhost"
   },
   "MessageService.API": {
     "AppId": "service-message-api"
   }
 }
```


#### HTTP 服务端实现

`[Get]`方法中请求DTO参数必须写`[FromQuery]`，不然会认为有`Body`而调用失败