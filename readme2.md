# freqtrade-console-stats
https://github.com/freqtrade/freqtrade/issues/6130#issuecomment-1170994704
How to implement the Fear & Greed Index?
```
def fear_index(self, dataframe, feardf):
        prev_resp = {
            "name": "Fear and Greed Index",
            "data": [
                {
                    "value": "3",
                    "value_classification": "Neutral",
                    "timestamp": str(datetime.today()),
                }
            ]
        }

        if feardf.empty:
            resp = requests.get('https://api.alternative.me/fng/?limit=9999&date_format=kr')
        else:
            if datetime.today() in feardf['date'].values:
                return feardf['value_classification']
            else:
                resp = requests.get('https://api.alternative.me/fng/?limit=1&date_format=kr')

        if resp is not None and resp.headers.get('Content-Type').startswith('application/json'):
            try:
                self.prev_resp = resp.json()
                df_gf = self.prev_resp['data']
            except:
                self.prev_resp = prev_resp
                df_gf = self.prev_resp['data']
        else:
            self.prev_resp = prev_resp
            df_gf = self.prev_resp['data']

        fear = pd.json_normalize(df_gf)
        fear.rename(columns={'value': 'fear'}, inplace=True)
        fear.rename(columns={'timestamp': 'date'}, inplace=True)

        fear['value_classification'] = fear['value_classification'].replace('Extreme Fear', 5)
        fear['value_classification'] = fear['value_classification'].replace('Fear', 4)
        fear['value_classification'] = fear['value_classification'].replace('Neutral', 3)
        fear['value_classification'] = fear['value_classification'].replace('Greed', 2)
        fear['value_classification'] = fear['value_classification'].replace('Extreme Greed', 1)

        df_copy = dataframe[['date']].copy()
        df_copy["date"] = pd.to_datetime(df_copy["date"], unit='ms')
        df_copy['date'] = df_copy['date'].astype(str)
        df_copy['date'] = df_copy['date'].str[:10]
        feardf = pd.merge(df_copy, fear, on='date')

        return feardf['value_classification']
```