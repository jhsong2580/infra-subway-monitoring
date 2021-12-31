### 실행 스크립트

```
import http from 'k6/http';
import { check, group, sleep, fail } from 'k6';

export let options = {
  stages: [
    {duration: '1m', target: 50},
    {duration: '1m10s', target: 100},
    {duration: '10s', target: 0},
  ],
  thresholds: {
          http_req_duration: ['p(99)<100'], // 99% of requests must complete below 1.5s
        },
};

const BASE_URL = 'https://eedys-tomcat.p-e.kr'
const USERNAME = 'test@test.com';
const PASSWORD = 'test1234';

export default function ()  {

    var payload = JSON.stringify({
          email: USERNAME,
          password: PASSWORD,
        });

    var params = {
          headers: {
            'Content-Type': 'application/json',
          },
        };


    let loginRes = http.post(`${BASE_URL}/login/token`, payload, params);

    check(loginRes, {
          'logged in successfully': (resp) => resp.json('accessToken') !== '',
        });


    let authHeaders = {
          headers: {
            Authorization: `Bearer ${loginRes.json('accessToken')}`,
          },
        };

    let myObjects = http.get(`${BASE_URL}/members/me`, authHeaders).json();
    check(myObjects, { 'retrieved member': (obj) => obj.id != 0 });
    sleep(1);
};

```



### 결과 스크립트

```

          /\      |‾‾| /‾‾/   /‾‾/   
     /\  /  \     |  |/  /   /  /    
    /  \/    \    |     (   /   ‾‾\  
   /          \   |  |\  \ |  (‾)  | 
  / __________ \  |__| \__\ \_____/ .io

  execution: local
     script: ./frequent_page_load.js
     output: -

  scenarios: (100.00%) 1 scenario, 100 max VUs, 2m50s max duration (incl. graceful stop):
           * default: Up to 100 looping VUs for 2m20s over 3 stages (gracefulRampDown: 30s, gracefulStop: 30s)


     ✓ logged in successfully
     ✓ retrieved member

     checks.........................: 100.00% ✓ 14310      ✗ 0    
     data_received..................: 5.4 MB  38 kB/s
     data_sent......................: 3.7 MB  26 kB/s
     http_req_blocked...............: avg=50.4µs   min=3.54µs  med=6.18µs  max=63.57ms p(90)=9.66µs   p(95)=20.93µs 
     http_req_connecting............: avg=4.38µs   min=0s      med=0s      max=2.05ms  p(90)=0s       p(95)=0s      
   ✓ http_req_duration..............: avg=7.38ms   min=2.98ms  med=5.82ms  max=64.06ms p(90)=12.52ms  p(95)=16.23ms 
       { expected_response:true }...: avg=7.38ms   min=2.98ms  med=5.82ms  max=64.06ms p(90)=12.52ms  p(95)=16.23ms 
     http_req_failed................: 0.00%   ✓ 0          ✗ 14310
     http_req_receiving.............: avg=105.88µs min=26.33µs med=66.18µs max=18.51ms p(90)=157.02µs p(95)=257.24µs
     http_req_sending...............: avg=43.12µs  min=10.7µs  med=21.5µs  max=7.21ms  p(90)=50.63µs  p(95)=90.88µs 
     http_req_tls_handshaking.......: avg=31.06µs  min=0s      med=0s      max=47.19ms p(90)=0s       p(95)=0s      
     http_req_waiting...............: avg=7.23ms   min=2.88ms  med=5.65ms  max=63.93ms p(90)=12.31ms  p(95)=16.09ms 
     http_reqs......................: 14310   101.727717/s
     iteration_duration.............: avg=1.01s    min=1s      med=1.01s   max=1.1s    p(90)=1.02s    p(95)=1.03s   
     iterations.....................: 7155    50.863858/s
     vus............................: 5       min=1        max=99 
     vus_max........................: 100     min=100      max=100

```