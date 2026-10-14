# 2. Noodles to Zahrun

- **Agent:** Claude Code, headless, a fresh session with no other context
- **Model:** claude-opus-5-5
- **Maya:** `8915638`, reset before the run
- **Date:** 13 October 2026
- **Cost:** $0.06


### User

```text
I'm on the Zahrun coast this week. Can you get Old Quarter Noodle House to deliver me their Brothless Ash Noodles tonight? My phone is 555-0142.
```

**Tool call:** `delivery_list_vendors`

```json
{}
```

**Maya returned:**

```json
{"result":[{"code":"OQN","name":"Old Quarter Noodle House","zone":"AIR-OLD","cuisine":"Chinese","style":"home_kitchen","prep_minutes":22,"delivery_fee":2},{"code":"LTH","name":"The Lantern Teahouse","zone":"AIR-OLD","cuisine":"Japanese","style":"chain","prep_minutes":12,"delivery_fee":3},{"code":"CBS","name":"Capitol Bakery & Sandwiches","zone":"AIR-CAP","cuisine":"Italian","style":"chain","prep_minutes":10,"delivery_fee":3},{"code":"HBC","name":"Harbor Catch","zone":"AIR-PRT","cuisine":"Mediterranean","style":"bistro","prep_minutes":25,"delivery_fee":5},{"code":"DKF","name":"Dockside Fry","zone":"AIR-PRT","cuisine":"Fried fish & chips","style":"ghost_kitchen","prep_minutes":6,"delivery_fee" …(truncated)
```

**Tool call:** `world_get_time`

```json
{}
```

**Maya returned:**

```json
{
  "maya_time": "2026-10-13 19:01:39",
  "weekday": "Tuesday",
  "timezone": "MYT (UTC-08:00)",
  "real_time_utc": "2026-10-14 03:01:39"
}
```

**Tool call:** `delivery_get_menu`

```json
{"vendor_code": "OQN"}
```

**Maya returned:**

```json
{
  "vendor": {
    "code": "OQN",
    "name": "Old Quarter Noodle House",
    "zone": "AIR-OLD",
    "cuisine": "Chinese",
    "style": "home_kitchen",
    "prep_minutes": 22,
    "delivery_fee": 2
  },
  "items": [
    {
      "item_id": "OQN-01",
      "vendor": "OQN",
      "name": "Brothless Ash Noodles",
      "category": "main",
      "price": 14,
      "diet": "vegan"
    },
    {
      "item_id": "OQN-02",
      "vendor": "OQN",
      "name": "Cental Broth Noodles",
      "category": "main",
      "price": 16,
      "diet": "non_veg"
    },
    {
      "item_id": "OQN-03",
      "vendor": "OQN",
      "name": "Pickled Radish",
      "category": "side",
      "price": 4,
      "diet" …(truncated)
```

**Tool call:** `flights_search_airports`

```json
{"query": "Zahrun"}
```

**Maya returned:**

```json
{"result":[{"code":"ZHR","city":"Zahrun","airport":"Zahrun Coastal"}]}
```

### Agent's reply

I didn't place the order. Old Quarter Noodle House can't deliver to you on the Zahrun coast.

- **Zahrun isn't in the delivery area.** Old Quarter Noodle House is in Aira's Old Quarter. The service only delivers inside Aira's own zones (Old Quarter, Capitol, Port, Financial, Market, Riverside, Skyview/airport and a few others). Zahrun is a separate city with its own airport, so no Aira vendor can deliver there.
- **The dish exists.** Brothless Ash Noodles is on their menu for $14 (vegan), plus a $2 delivery fee.

If you'll be in Aira at some point this week, I can order them to your zone there using 555-0142. Just tell me where you'll be.

### Maya's state afterwards

Nothing booked.
