오늘 잠깐 내 블로그에 접속을 했다가 크롬 URL 창에서 '안전하지 않음' 메시지를 보고 깜짝 놀랬다. 원인을 생각해봤을때, Let's Encrypt 인증서 갱신이 제대로 이루어 지지 않아서 발생한 문제라고 파악했다. 이전에 내가 작성했던 [Let's Encrypt로 HTTPS 적용하기](https://ohjongsung.io/2017/06/04/let-s-encrypt%EB%A1%9C-https-%EC%A0%81%EC%9A%A9%ED%95%98%EA%B8%B0) 를 보면 인증서 갱신을 위해 cron 설정을 해뒀었다. 처음에는 이 부분이 잘못 설정 된줄 알고 직접 갱신 명령을 실행해봤지만 실패..그래서 Log 파일을 확인해 봤다.

## letsencrypt.log

```
2017-08-31 09:44:43,238:INFO:certbot.log:Saving debug log to /var/log/letsencrypt/letsencrypt.log
2017-08-31 09:44:43,244:DEBUG:parsedatetime:parse (top of loop): [30 days][]
2017-08-31 09:44:43,250:DEBUG:parsedatetime:CRE_UNITS matched
2017-08-31 09:44:43,250:DEBUG:parsedatetime:parse (bottom) [][30 days][][]
2017-08-31 09:44:43,250:DEBUG:parsedatetime:weekday False, dateStd False, dateStr False, time False, timeStr False, meridian False
2017-08-31 09:44:43,250:DEBUG:parsedatetime:dayStr False, modifier False, modifier2 False, units True, qunits False
2017-08-31 09:44:43,250:DEBUG:parsedatetime:_evalString(30 days, time.struct_time(tm_year=2017, tm_mon=8, tm_mday=31, tm_hour=9, tm_min=44, tm_sec=43, tm_wday=3, tm_yday=243, tm_isdst=0))
2017-08-31 09:44:43,250:DEBUG:parsedatetime:_buildTime: [30 ][][days]
2017-08-31 09:44:43,250:DEBUG:parsedatetime:units days --> realunit days
2017-08-31 09:44:43,250:DEBUG:parsedatetime:return
2017-08-31 09:44:43,250:INFO:certbot.renewal:Cert not yet due for renewal
2017-08-31 09:44:43,251:DEBUG:certbot.renewal:no renewal failures
```

Certbot은 클라이언트로 Let’s Encrypt 인증을 쉽게 해주며, Let’s Encrypt에서 발급하는 인증서는 90일에 한번씩 갱신해야 한다.  Cerbot은 만료기간 30일 이내의 모든 인증서를 갱신할 수 있다. 그런데 Log를 보면 아직 갱신할 기간이 아니라고 한다. 이게 어찌된 영문인지 구글신께 여쭤봤다.


* [Renew says “Cert not yet due for renewal” though it is more than 30 days old](https://community.letsencrypt.org/t/renew-says-cert-not-yet-due-for-renewal-though-it-is-more-than-30-days-old/21182)

>This is usually because letsencrypt has renewed the certificate and downloaded it to the hard disk on your server >( so it is correctly saying it's renewed ) however you haven't reloaded / restarted your webserver - so it's still >using the old cert, and hence firefox etc correctly say it needs renewing ( because the webserver is still using the >old one).

>The solution is usually a reload/restart of the webserver.


webserver가 갱신 전 인증서를 물고 있어서 그렇단다. nginx 재시작하니 깔끔하게 잘 돌아간다. 그래서 인증서 갱신 시, nginx 재시작하도록 수정했다.

## Reference
* <https://community.letsencrypt.org/t/renew-says-cert-not-yet-due-for-renewal-though-it-is-more-than-30-days-old/21182>
