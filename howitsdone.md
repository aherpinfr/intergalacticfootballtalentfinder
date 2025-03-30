# How the Dashboard is built ?

The dashboard is based on the dataset "datasetpotentialplayers" that has been 
created for the purpose of this project

![image](https://github.com/user-attachments/assets/1ceef3a0-49c4-4535-bdbb-132ca1583047)

Two table were created ex nihilo (dim_clubs an dim_players_characteristics) and imported as csv in BigQuery.

Dim_position_type was then created because the position type was lacking in dim_players_characteristics.
![image](https://github.com/user-attachments/assets/2b0374a6-ce2f-4665-a8d8-75fd5c67c824)

In order to display the best opportunities for a given club two view where then created, dim_standards and dim_players_opportunities.

All the recommendations are coming from dim_players_opportunities.
![image](https://github.com/user-attachments/assets/d5fb42b9-96aa-4647-a399-8d70f6fa42d0)

Recommended players for Nova FC.

# What is the point of dim_standards ?

In order to make recommendation for a given club, I needed first to define criteria of what a good recommendation is for the selected club.

A good recommendation follows the following criteria:

- First, the recommended player's value must be equal or cheaper than the club budget
- Second, the recommended player's potential must be equals or the superior to the current rating standard of the selected club, and for the selected position type. 

Hence the point of dim_standards.

## How dim_standards was built ?

First, I defined the following standards :
- For attacker : A club's standard is equal to the second higest rated attacker of the selected club
- For midfielder/defender : A club's standard  is equal to the third higest rated mid/def of the selected club
- For Goalkeeper : A club's standard  is equal to the first higest rated goalkeeper of the selected club

This then has been translated in the SQL instructions as below

```sql
With Subquery AS (SELECT Club, id_club, id_player, overall_rating, dpt.string_field_1 as position_type,
ROW_NUMBER() OVER(PARTITION BY club, dpt.string_field_1 order by overall_rating desc) as rank_overall
from `datasetpotentialplayers.dim_players_characteristics` as dpc
left join `datasetpotentialplayers.dim_position_type` as dpt
on dpc.position = dpt.string_field_0)

SELECT club,  overall_rating, position_type, rank_overall, id_club, id_player from Subquery
where (position_type = 'GK'and rank_overall = 1) 
or (position_type = 'ATT' and rank_overall = 2)
or (position_type = 'DEF' and rank_overall = 3) 
or (position_type = 'MID' and rank_overall = 3)
order by club, position_type
```
Concept of the query : All players are selected and they are ranked on their rating thanks to ROW_NUMBER after having been partitioned by their clubs and their position type.

The subquery is then filtered for each club on the corresponding rank I have defined as the standards.

Here are the following results for the club Alpha Rangers.
![image](https://github.com/user-attachments/assets/23c40afd-9670-4f75-bb2f-d51c67ab073f)

The results will be then used to create the final table dim_opportunities




