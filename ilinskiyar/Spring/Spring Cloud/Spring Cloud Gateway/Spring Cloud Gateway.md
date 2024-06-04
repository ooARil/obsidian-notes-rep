Spring cloud Gateway - реализует Api Gateway, т.е. это единая точка входа для всех наших сервисов. Т.е. мы можем создать один сервис, который будет транслировать все вызовы к нашим сервисам (которые могут быть даже не опубликованы, т.е. очень похож на nginx).

Вот пример запроса, который будет работать

curl --location --request POST 'http://localhost:8081/api/login?username=test&password=test' \
--header 'Host: localhost:8094' \
--header 'sec-ch-ua: "Chromium";v="122", "Not(A:Brand";v="24", "YaBrowser";v="24.4", "Yowser";v="2.5"' \
--header 'Accept: application/json, text/plain, */*' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--header 'Referer: http://localhost:8093/' \
--header 'sec-ch-ua-mobile: ?0' \
--header 'User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 YaBrowser/24.4.0.0 Safari/537.36' \
--header 'sec-ch-ua-platform: "Windows"'