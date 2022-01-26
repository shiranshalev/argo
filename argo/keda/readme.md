# pif debugging the scaler..

curl -X POST -H "Content-Type: application/json" --data '{ "query": "{grid{uri}}" }' -s http://localhost:4444/graphql | jq .
curl -X POST -H "Content-Type: application/json" --data '{ "query": "{sessionsInfo{sessionQueueRequests}}" }' -s http://localhost:4444/graphql | jq .

curl -X POST -H "Content-Type: application/json" --data '{"query": "{ sessionsInfo { sessionQueueRequests, sessions { id, capabilities, nodeId } } }" }' -s http://localhost:4444/graphql | jq .