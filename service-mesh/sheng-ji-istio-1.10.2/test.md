# test

productpage 中测试 

```text
curl ${REVIEWS_SERVICE_HOST}:${REVIEWS_SERVICE_PORT_HTTP}/reviews/0
```

对应的测试结果

```text
## v1
curl ${REVIEWS_SERVICE_HOST}:${REVIEWS_SERVICE_PORT_HTTP}/reviews/0 
{"id": "0","reviews": [{  "reviewer": "Reviewer1",  "text": "An extremely entertaining play by Shakespeare. The slapstick humour is refreshing!", "rating": {"stars": 5, "color": "red"}},{  "reviewer": "Reviewer2",  "text": "Absolutely fun and entertaining. The play lacks thematic depth when compared to other plays by Shakespeare.", "rating": {"stars": 4, "color": "red"}}]}

## v2
istio-proxy@productpage-v1-5d9b4c9849-x7f8h:/$ curl ${REVIEWS_SERVICE_HOST}:${REVIEWS_SERVICE_PORT_HTTP}/reviews/0 
{"id": "0","reviews": [{  "reviewer": "Reviewer1",  "text": "An extremely entertaining play by Shakespeare. The slapstick humour is refreshing!", "rating": {"stars": 5, "color": "black"}},{  "reviewer": "Reviewer2",  "text": "Absolutely fun and entertaining. The play lacks thematic depth when compared to other plays by Shakespeare.", "rating": {"stars": 4, "color": "black"}}]}

## v3
istio-proxy@productpage-v1-5d9b4c9849-x7f8h:/$ curl ${REVIEWS_SERVICE_HOST}:${REVIEWS_SERVICE_PORT_HTTP}/reviews/0 
{"id": "0","reviews": [{  "reviewer": "Reviewer1",  "text": "An extremely entertaining play by Shakespeare. The slapstick humour is refreshing!"},{  "reviewer": "Reviewer2",  "text": "Absolutely fun and entertaining. The play lacks thematic depth when compared to other plays by Shakespeare."}]}

```

