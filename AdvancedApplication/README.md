# 進階應用
* 服務探針

## [服務探針]
### livenessProbe: 定期向容器發送HTTP / TCP / 命令請求，檢查容器內的服務是否正常，如回應失敗時，會將容器重啟。
### startupProbe: 用於檢查容器是否啟動完畢，如還未啟動完成，此容器服務不會放行出去。
