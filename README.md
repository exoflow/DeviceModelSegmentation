# Mobile Device Model Segmentation

## What’s this about?
This approach may not only protect from wasting advertising money but also enable to profitably scale into untapped audiences. It is important to understand that the goal of this device model segmentation is not to identify the high value devices but rather to create device model segments which can be used in parallel for more detailed targeting and true value bidding.

## Calculate the device value and create segments
Firstly, why do device models have a different value? The assumption is that there is a correlation between the device model and the average value of the users that owns this device. In practice this is certainly true. Let’s define 3 segments for our device models:
A — The few devices which belong to high value users.
B — The middle segment.
C — The “long tail”: Many devices with a low value per device.

``` python
import pandas as pd
# Load installs.csv
installs = pd.read_csv('https://raw.githubusercontent.com/exoflow/DeviceModelSegmentation/master/installs.csv', sep = ';')
installs
# Load revenue.csv
revenue = pd.read_csv('https://raw.githubusercontent.com/exoflow/DeviceModelSegmentation/master/revenue.csv', sep = ';')
revenue
# Merge them together
merged = installs.merge(revenue)
merged
# Make a pivot table and group per device
groupby = merged.groupby('Device')['Revenue'].agg({'count','sum'}).rename(columns={'count':'Total Users', 'sum':'Total Revenue'})
# Calculate the Revenue per User column
groupby['Revenue per User'] = groupby['Total Revenue']/groupby['Total Users']
# Assign Device segments
groupby['Device Segment'] = pd.qcut(groupby['Revenue per User'], 3, labels=["C", "B", "A"])
groupby.sort_values('Revenue per User', ascending = False)
```

Thats really all it is. Note that in this example I used the Pandas qcut() function which creates segments of equal size. One could also use segments based on percentiles. 
Upload the device segments
This is the final step to get value out of the device segmentation. Most mobile advertising partners offer self-serve dashboards or APIs where advertisers can upload their device segments. Here are two exemplary script for Facebook Ads and Applovin (other channels have similar features).

Facebook Ads Advanced Targeting API
Source: https://developers.facebook.com/docs/marketing-api/audiences/reference/advanced-targeting/
Example Request:

``` javascript
curl \
  -F 'name=My AdSet' \
  -F 'optimization_goal=REACH' \
  -F 'billing_event=IMPRESSIONS' \
  -F 'bid_amount=2' \
  -F 'daily_budget=1000' \
  -F 'campaign_id=<CAMPAIGN_ID>' \
  -F 'targeting={ 
    "geo_locations": {"countries":["US"]}, 
    "user_device": ["Galaxy S6","One m9"], 
    "user_os": ["android"] 
  }' \
  -F 'status=ACTIVE' \
  -F 'access_token=<ACCESS_TOKEN>' \
  https://graph.facebook.com/v2.11/act_<AD_ACCOUNT_ID>/adsets
  ```

Applovin Device Model Targeting API
Source: https://growth-support.applovin.com/hc/en-us/articles/115000788668-Device-Model-Targeting-API
Example Request:

``` javascript
Example Request:
curl -X POST --data-binary @devices.txt "https://api.applovin.com/devices/append?device_list_id=<DEVICE_LIST_ID>&api_key=<API_KEY>"
Where the contents of the file devices.txt is:
SM-J100H
HUAWEI Y520-U03
Example Response:
{ "device_list_id" : “dab2d1297d35592597b0eee016e92baa”, "processed_devices" : 2, "failed_devices" : 0 }
```

Once the device segments are uploaded the campaign can be split into multiple smaller campaigns, each one targeting exactly one device segment. With this campaign setup it is more likely to get a good ROAS on all of these segments as the bid amount will now closer reflect the true value of the audience.

## Collect device model information
Commercial model names of devices like for example “SM-G950F” (Samsung Galaxy S8) or “iPhone6,1” are available through many sources. Mobile attribution providers like Appsflyer and Adjust include device models in their raw data reports and popular analytics SDKs like Unity Analytics and Firebase automatically collect this information.

## Pros and Cons
On the pro side this approach can yield a higher ROAS because the bid amount better reflects the real value of the targeted audience. Bidding high for top devices can lead to a competitive advantage over other advertisers which have a flat bid across all devices. At the same time, the potential scale of campaigns is high because the audience is not limited as device models are not blacklisted.
However, the advertising products move more into autopilot features like the Google UAC where the advertisers’ choices are limited. Also for Facebook Ads it might work better to not limit Facebooks bidding algorithm but rather let Facebook decide which devices to target at which bid (e.g. by using App Event Optimization).
In the end the key is to try and learn what works best. That’s the only way to run successful mobile advertising campaigns in the long run.

