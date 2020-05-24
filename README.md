# GORM Log adds context features

> If I want to set up a separate context context for a HTTP session request, You can use this feature.

- Testing steps
- export DEBUG=true
- export LOGCTX=true
- go test -v -run "TestLogger"

## 使用描述
这个类库主要功能是对GORM的请求设置上下文信息，使用方法：

设置上下文：

```
DB.SetCtx(ctxInfo)
```

获取上下文信息：

```
//GET context information
ctxTmp,_:=builder.GetCtx()
ctxInfo2,_:=ctxTmp.(string)
```

## 官方方法参考
其他使用方法与GORM官网没有差异，如果不使用此功能可以直接用官方类库，其他gorm语法可以参考：

https://gorm.io/zh_CN/docs/index.html

## 使用举例
go_gateway 引入了 golang_common，golang_common 引用了该类包。所以go_gateway间接引用了本类库，使用它我们做了一下trace日志记录功能。
```
dbgorm.SetLogger(&MysqlGormLogger{Trace: NewTrace()})
// LogCtx(true) 时会执行改方法
func (logger *MysqlGormLogger) CtxPrint(s *gorm.DB,values ...interface{}) {
  ctx,ok:=s.GetCtx()
  trace:=NewTrace()
  if ok{
     trace=ctx.(*TraceContext)
  }
  message := logger.LogFormatter(values...)
  if message["level"] == "sql" {
     Log.TagInfo(trace, "_com_mysql_success", message)
  } else {
     Log.TagInfo(trace, "_com_mysql_failure", message)
  }
}
```
具体可以参见：https://github.com/e421083458/golang_common/blob/master/lib/mysql.go

最终达成的效果为，我们go_gateway项目中的日志可以打印效果出sql来，并且与请求的trace一致：
```
[INFO][2020-05-17T16:10:10.008][log.go:58] _com_mysql_success||level=sql||proc_time=0.000000000||affected_row=0||traceid=c0a803045ec0f161b1005f98104dc7b0||spanid=9e68f265380704bb||source=/Users/niuyufu/go/src/github.com/e421083458/go_gateway_demo/dao/service_info.go:92||current_time=2020-05-17 16:10:10||sql=SELECT count(*) FROM `gateway_service_info`  WHERE (is_delete=0) LIMIT 99999 OFFSET 0||cspanid=
```