# cryptocurrency-prices-grafana-redis-prometheus-flask-api

# 🚀 Fetching Cryptocurrency Prices from Coingecko and Displaying them on Grafana 🚀

https://github.com/coding-to-music/cryptocurrency-prices-grafana-redis-prometheus-flask-api

From / By Ruan Bekker https://github.com/ruanbekker

https://github.com/ruanbekker/cryptocurrency-prices-grafana

## Environment variables:

```java

```

## GitHub

```java
git init
git add .
git remote remove origin
git commit -m "first commit"
git branch -M main
git remote add origin git@github.com:coding-to-music/cryptocurrency-prices-grafana-redis-prometheus-flask-api.git
git push -u origin main
```

## Results

Pushgateway http://localhost:9092

Grafana http://localhost:3000/datasources

Prometheus http://localhost:35231/graph?g0.expr=cryptocurrency_price%7Bprovider%3D%22coingecko%22%7D&g0.tab=1&g0.stacked=0&g0.show_exemplars=0&g0.range_input=1h

```
docker-compose stop
```

Output:

```
Stopping grafana       ... done
Stopping prometheus    ... done
Stopping rates-shipper ... done
Stopping coins-api     ... done
Stopping pushgateway   ... done
Stopping rates-cache   ... done
```

# cryptocurrency-prices-grafana

Fetching Cryptocurrency Prices from Coingecko and Displaying them on Grafana

## About

This stack consists of:

- Prometheus (timeseries database)
- Pushgateway (introduces a /metrics endpoint for prometheus to scrape)
- Grafana (the best)
- Redis (k/v in memory store)
- Python Script that pulls crypto market data, publishes to pushgateway for prometheus to scrape, and stores market data in Redis
- Flask API to fetch market prices for other projects

I'm using coingecko's api which gives me 50 free calls per minute.

## Walkthrough

Clone the repo:

```
git clone https://github.com/ruanbekker/cryptocurrency-prices-grafana
```

Change to the directory and build/start the containers:

```
cd cryptocurrency-prices-grafana
docker-compose up -d --build
```

Accessing Grafana, for me its locally on http://grafana.127.0.0.1.nip.io:3000

![image](https://user-images.githubusercontent.com/567298/145399454-1a24cd8c-8482-469f-aff4-9d570c45667f.png)

Using username admin and password admin.

Accessing Prometheus on http://prometheus.127.0.0.1.nip.io:9090, we can see we can fetch our metrics using promql:

- `cryptocurrency_price{provider="coingecko"}`

![image](https://user-images.githubusercontent.com/567298/145400449-2410efcb-ffc0-48d8-879c-56f9d547391a.png)

When we access Pushgateway on http://pushgateway.127.0.0.1.nip.io:9091, we can see that prometheus is scraping pushgateway correctly:

![image](https://user-images.githubusercontent.com/567298/145400545-c59e09de-6294-4338-ac3a-c1ad4ce6e2e1.png)

When we create a Grafana Dashboard:

![image](https://user-images.githubusercontent.com/567298/145400814-161722c9-0b81-4085-a459-3d017cb1ef66.png)

- Query: `cryptocurrency_price{provider="coingecko"}`
- Legend: `{{coin}}`

To add dashboard variables: Settings -> Variables -> Add New Variable

![image](https://user-images.githubusercontent.com/567298/145401078-36901d9d-f60f-4907-9a65-08e54bbbceda.png)

Head back to the dashboard and update your query to include the coin that you select from the dropdown at the top, which will be referenced as the variable:

```
cryptocurrency_price{provider="coingecko", coin=~"$coin"}
```

Which will introduce this:

![image](https://user-images.githubusercontent.com/567298/145401631-fe739c10-854c-4bde-80cd-88aceab98b04.png)

Then it should look like this:

![image](https://user-images.githubusercontent.com/567298/145401269-0e011682-f186-4d09-97ce-88ca1b2b26d3.png)

After some customization:

![image](https://user-images.githubusercontent.com/567298/145401544-5ecee8a7-406a-45c9-abb5-87f69a308c54.png)

Coingecko allows 50 calls a minute, so for my own use-case I am fetching the data once a minute and store it in the redis cache, so that the api fetches the data from cache, so I can call my api as much as I want, the only downside is that the data will be a minute old.

We can see from redis that our data was cached:

```
docker exec -it rates-cache sh -c "redis-cli -n 1 keys \*"
1) "MATIC_TO_USD"
2) "LINK_TO_USD"
3) "DOGE_TO_USD"
4) "XRP_TO_USD"
5) "ADA_TO_USD"
6) "VET_TO_USD"
7) "TRX_TO_USD"
8) "ETH_TO_USD"
9) "BTC_TO_USD"
```

And to get the value of BTC in USD:

```
docker exec -it rates-cache sh -c "redis-cli -n 1 get BTC_TO_USD"
"49272"
```

Our Flask API uses the redis cache to fetch the values, and can be accessed like this:

```
curl http://api.127.0.0.1.nip.io:5000/coins/btc
{"acronymm":"BTC","current_price_in_usd":"49533"}

curl http://api.127.0.0.1.nip.io:5000/coins/matic
{"acronymm":"MATIC","current_price_in_usd":"2.32"}
```
