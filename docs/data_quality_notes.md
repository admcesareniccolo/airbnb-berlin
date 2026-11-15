#### DATASET DATA QUALITY ANALYSIS

##### LISTINGS | listings.csv.gz

###### NULL columns in listings.csv.gz
- neighborhood_overview
- host_since
- host_response_time
- host_response_rate 
- host_acceptance_rate
- host_thumbnail_url
- host_neighbourhood 
- host_total_listings_count
- host_verifications
- neighbourhood
- bathrooms
- calendar_updated
- instant_bookable

###### Decision on NULL colunns and columns which are useful for my analysis
I decided to distribute the columns in three lists in the **/notebooks/01_exploration.ipynb** notebook:

keep_col = ['id', 'host_id', 'name', 'host_listings_count', 'calculated_host_listings_count',
 'neighbourhood_cleansed', 'neighbourhood_group_cleansed', 'latitude', 'longitude',
 'property_type', 'room_type', 'accommodates', 'bedrooms', 'beds',
 'price_quote_total_price', 'price_quote_price_per_night', 'minimum_nights', 'maximum_nights',
 'availability_365', 'availability_eoy', 'number_of_reviews', 'number_of_reviews_ltm',
 'estimated_occupancy_l365d', 'estimated_revenue_l365d',
 'review_scores_rating', 'review_scores_accuracy', 'review_scores_cleanliness',
 'review_scores_checkin', 'review_scores_communication', 'review_scores_location', 'review_scores_value']

transform_col = ['price', 'price_quote_checkin_date', 'price_quote_checkout_date', 'price_quote_raw',
 'hosts_time_as_host_years', 'hosts_time_as_host_months', 'host_is_superhost',
 'host_has_profile_pic', 'host_identity_verified', 'has_availability',
 'bathrooms_text', 'amenities', 'first_review', 'last_review', 'license', 'reviews_per_month']

drop_col = ['listing_url', 'scrape_id', 'last_scraped', 'source', 'description', 'neighborhood_overview',
 'picture_url', 'host_url', 'host_profile_id', 'host_profile_url', 'host_name', 'host_since',
 'hosts_time_as_user_years', 'hosts_time_as_user_months', 'host_location', 'host_about',
 'host_response_time', 'host_response_rate', 'host_acceptance_rate', 'host_thumbnail_url',
 'host_picture_url', 'host_neighbourhood', 'host_total_listings_count', 'host_verifications',
 'neighbourhood', 'bathrooms', 'calendar_updated', 'instant_bookable',
 'minimum_minimum_nights', 'maximum_minimum_nights', 'minimum_maximum_nights',
 'maximum_maximum_nights', 'minimum_nights_avg_ntm', 'maximum_nights_avg_ntm',
 'availability_30', 'availability_60', 'availability_90', 'calendar_last_scraped',
 'number_of_reviews_l30d', 'number_of_reviews_ly',
 'calculated_host_listings_count_entire_homes', 'calculated_host_listings_count_private_rooms',
 'calculated_host_listings_count_shared_rooms']

###### Host listing counts: inconsistency between 'host_listings_count' and 'calculated_host_listings_count'

**Context.** `host_listings_count` (host-level count, likely from the host profile) and `calculated_host_listings_count` (computed by Inside Airbnb from the listings in this snapshot) are expected to be related: the host-level count should never be lower than the number of listings actually observed for that host in the dataset.

**Findings** (snapshot 2026-09-19, 12,376 listings)
| Relationship | Rows | Share |
|---|---|---|
| `host_listings_count` = `calculated_host_listings_count` | 9,917 | 80.1% |
| `host_listings_count` > `calculated_host_listings_count` | 2,454 | 19.8% |
| `host_listings_count` < `calculated_host_listings_count` | 5 | 0.04% |

**Issue.** In 5 listings the host-level count is lower than the count observed in the dataset, which is logically inconsistent (a host cannot have fewer listings than we observe). Affected listing ids: `1562192543837144245`, `1562414364938331067`, `44483955`, `44872206`, `1633358579617436258`.

**Interpretation (hypothesis, not verified).** The 2,454 cases where the host-level count is higher may reflect listings outside Berlin or outside this snapshot. The 5 inconsistent cases may come from stale or out-of-sync host data.

**Decision.** Keep only `calculated_host_listings_count` in the core layer: its definition is consistent with the dataset, it has no inconsistencies, and it is identical to `host_listings_count` in 80% of rows. `host_listings_count` remains available in staging. The 5 records are retained and flagged here, not deleted.