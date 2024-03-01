## 10000 h1 disclosed reports

![image](https://github.com/shreyaschavhan/10000-h1-disclosed-reports/assets/68887544/9e6e812e-fdaa-48f7-9a39-06a1b4b26938)

---

On `31st Dec 2023`, I made it my goal to read `10,000 H1 Reports in 2024 Q1 (i.e. first 3 months)` to really understand deep down what kind of bugs are being reported, accepted, or rejected and how exactly 
I should approach my journey in #bugbounty. Also, I thought, there was no better resource than actual disclosed bug reports. Later I decided to cap my goal at `*5000*` because I think I nailed the common pattern and already accomplished what I wanted to get out of it.

It took me 9 weeks in total i.e. approx 60 days from 1st Jan to 28th Feb (or approx 40 days if I don't count holidays I took)

I read on average 125 reports per day.
Total Time Spent reading H1 reports: 98 hours 52 min

![image](https://github.com/shreyaschavhan/10000-h1-disclosed-reports/assets/68887544/a9727b89-6dcb-445e-ba99-106736280105)

---

I made a post on X sharing this here: https://twitter.com/shreyas_chavhan/status/1763032214508339701

and a few of you guys were asking me how they should collect these many reports, so I thought of sharing the reports that I collected myself. 

What I did was:
- Intercepted a request to H1 hacktivity
- As much as I remember, I either modified the limit parameter from 25 to 10,000 (that didn't work i think) or iterated through 1 to 10,000 with 25/50 step each to collect json response with every 25 or 50 h1 reports till I reach 10,000 (they don't allow more than 10,000 for some reason idk why).
- Processed that json response and collected report_id's
- Added `https://hackerone.com/reports/` prefix to add report ids
- That's the final python script:

```python
import requests
import json

# GraphQL query
j = 0
while j < 20000000:
    graphql_query = {
        "operationName": "HacktivitySearchQuery",
        "variables": {
            "queryString": "*:*",
            "size": 25,
            "from": 0 + j,
            "sort": {
                "field": "disclosed_at",
                "direction": "DESC"
            },
            "product_area": "hacktivity",
            "product_feature": "overview"
        },
        "query": "query HacktivitySearchQuery($queryString: String!, $from: Int, $size: Int, $sort: SortInput!) {\n me {\n id\n __typename\n }\n search(\n index: HacktivityReportIndexService\n query_string: $queryString\n from: $from\n size: $size\n sort: $sort\n ) {\n __typename\n total_count\n nodes {\n __typename\n ... on HacktivityReportDocument {\n id\n _id\n reporter {\n id\n name\n username\n ...UserLinkWithMiniProfile\n __typename\n }\n cve_ids\n cwe\n severity_rating\n upvoted: upvoted_by_current_user\n report {\n id\n databaseId: _id\n title\n substate\n url\n disclosed_at\n report_generated_content {\n id\n hacktivity_summary\n __typename\n }\n __typename\n }\n votes\n team {\n handle\n name\n medium_profile_picture: profile_picture(size: medium)\n url\n id\n currency\n ...TeamLinkWithMiniProfile\n __typename\n }\n total_awarded_amount\n latest_disclosable_action\n latest_disclosable_activity_at\n submitted_at\n __typename\n }\n }\n }\n}\n\nfragment UserLinkWithMiniProfile on User {\n id\n username\n __typename\n}\n\nfragment TeamLinkWithMiniProfile on Team {\n id\n handle\n name\n __typename\n}\n"
    }

    # URL of the GraphQL endpoint
    graphql_endpoint = 'https://hackerone.com/graphql'

    # Send POST request with GraphQL query
    response = requests.post(graphql_endpoint, json=graphql_query)

    # Print response content
    if response.status_code == 200:
        data = response.json()
        with open('h1reports.txt', 'a') as f:
            for i in data["data"]["search"]['nodes']:
                print(i['report']['url'])
                f.write(i['report']['url'] + '\n')
        
    else:
        print(f"Request failed with status code: {response.status_code}")

    j += 25
```






