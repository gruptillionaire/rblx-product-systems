# ROBLOX Product & Systems Work

I have spent several years building and operating Roblox-based live consumer products. As succommercial/live projects, the source is private -- this page focuses on product outcomes, technical systems, and design tradeoffs.
A fast iterative design process as well as long-term involvement on the platform has led to an overwhelming amount of unique systems as candidates to be shown. The following is a only very small percentage of developed products.
The following projects were all made solo, with at most one contractor.

## Scale / Product Context

- Reached 6M+ visits and 2,000+ concurrent users across shipped products
- Owned product design, implementation, analytics, live operations, user feedback, economy / progression tuning and curves

## Project: Infinite Runner

<img width="1562" height="637" alt="image" src="https://github.com/user-attachments/assets/141e6bd0-af2c-4a40-8a93-4e2c3f269dae" />
<img width="1572" height="646" alt="image" src="https://github.com/user-attachments/assets/8f7fc459-526f-401a-a277-b8a38afea945" />

- An infinite runner game wherein a player runs as far as they can with their energy, then uses money obtained by distance ran to upgrade speed and maximum energy.
- 2,000 concurrent users peak
- Popularity lifespan of ~a month (which is high!) then a revival at the Christmas update released
<img width="1470" height="573" alt="image" src="https://github.com/user-attachments/assets/19c32c4c-8455-4962-82c6-01f629d187ba" />

Initial metrics were quite poor. I took user feedback into account, ran mathematical models to curve progression to be more responsive, leading to a rise in average playtime by about 50%, from 9 minutes to a solid 15 minutes, which it currently remains on, and raising day 1 retention to a ~7% additive lift, from 5% to ~12%.
<img width="608" height="222" alt="image" src="https://github.com/user-attachments/assets/1587424c-57e9-4061-a632-d8110daedbf9" />
<img width="1482" height="575" alt="image" src="https://github.com/user-attachments/assets/45a83374-84c9-4159-967c-6526a77be2fb" />

(Retention analytic is buggy, but as one can obviously see a recent 14.29% user retention would put the game in the top 90th percentile)

### Features:
- Infinite procedurally generated map generated pseudorandomly each life
- Client-side exploit-safe rare spawn pickups validated through the pseudorandom seed generator
- Safe rendering distance for lower-end mobile devices
- Data saving inventory, unique unlockable abilities, daily challenges, permanent upgrades

### Map
An infinitely generating, client-unique map is not an easy task to do. If we were to make the server authoritate the map then the user would not be able to generate a new map per life (because of course it would be tracking the life states of every single player), but without any server validation then a bad actor would be able to access the entire game and get an unlimited high score. Utilising a server-authoritated seed for each player per life, the player could reconstruct their map on their local device and any relevant pickups could be validated by the current chunk.
Technically this allows the client to enter a chunk, 'guess' the current pickups, then send it through without even collecting, but this was an acceptable risk -- pickups are obtainable by running over them, so the only challenge is getting to them. A bad actor would still have to get to the chunk to redeem a new pickup, so their benefit by cheating is miniscule.

### Summary
Unfortunately for this game - and I say this candidly - I was not interested in it. I could absolutely have extended its lifespan and thus monetisation to at least three months, but after creating the map generation it no longer interested me. The statistics are still quite impressive and well past the average game's benchmarks, so this experience could likely be revived back to the status quo if an interested party were to do so.

