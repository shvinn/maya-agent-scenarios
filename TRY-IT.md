# Try your agent in Maya

Ten prompts to paste into your AI agent (Claude, ChatGPT, Codex, Cursor or
anything else that speaks MCP), with the result a good agent reaches, so
you can see how yours does. Each prompt has a trap that a real service would
set. No code, no setup beyond connecting Maya.

1. [Run Maya](#1-run-maya)
2. [Connect your agent](#2-connect-your-agent)
3. [Try the prompts](#3-try-the-prompts)

## 1. Run Maya

With Docker:

```bash
docker run -p 127.0.0.1:6292:6292 -v maya-data:/data ghcr.io/shvinn/maya
```

Or from source: clone [Maya](https://github.com/shvinn/maya) and run
`uv run maya serve --domain all`.

Either way, Maya serves one MCP endpoint per service:

| Service | URL |
| --- | --- |
| Flights | `http://localhost:6292/flights/mcp` |
| Hotels | `http://localhost:6292/hotels/mcp` |
| Car rental | `http://localhost:6292/cars/mcp` |
| Food delivery | `http://localhost:6292/delivery/mcp` |
| Events | `http://localhost:6292/events/mcp` |

There's also `/world/mcp`, which controls Maya's clock and resets the world.
Don't give that one to your agent.

## 2. Connect your agent

Add the services a prompt needs, or all five. Each is a separate MCP server
over streamable HTTP, with no authentication.

### Agents on your machine

These can reach `localhost` directly.

**Claude Code**

```bash
claude mcp add --transport http maya-flights http://localhost:6292/flights/mcp
claude mcp add --transport http maya-hotels http://localhost:6292/hotels/mcp
claude mcp add --transport http maya-cars http://localhost:6292/cars/mcp
claude mcp add --transport http maya-delivery http://localhost:6292/delivery/mcp
claude mcp add --transport http maya-events http://localhost:6292/events/mcp
```

**Codex CLI**: in `~/.codex/config.toml`:

```toml
[mcp_servers.maya-flights]
url = "http://localhost:6292/flights/mcp"

[mcp_servers.maya-hotels]
url = "http://localhost:6292/hotels/mcp"

# ...and the same for cars, delivery and events
```

**Cursor**: in `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "maya-flights": { "url": "http://localhost:6292/flights/mcp" },
    "maya-hotels": { "url": "http://localhost:6292/hotels/mcp" },
    "maya-cars": { "url": "http://localhost:6292/cars/mcp" },
    "maya-delivery": { "url": "http://localhost:6292/delivery/mcp" },
    "maya-events": { "url": "http://localhost:6292/events/mcp" }
  }
}
```

**Claude Desktop**: in `claude_desktop_config.json`, bridging each URL with
`mcp-remote`:

```json
{
  "mcpServers": {
    "maya-flights": { "command": "npx", "args": ["mcp-remote", "http://localhost:6292/flights/mcp"] }
  }
}
```

### Agents in the cloud (ChatGPT, Claude.ai)

These connect from their own servers, so they can't reach your
`localhost`. Give Maya a temporary public HTTPS address with a tunnel, for
example [ngrok](https://ngrok.com):

```bash
ngrok http 6292 --host-header=rewrite
```

Keep `--host-header=rewrite`: without it, Maya started from source refuses
tunnelled requests. ngrok prints an address like
`https://something.ngrok-free.dev`. Your Maya URLs are then
`https://something.ngrok-free.dev/flights/mcp`, and so on.

Then add each URL as a remote MCP server:

- **ChatGPT**: turn on developer mode (Settings → Apps & Connectors →
  Advanced settings), then create a connector for each URL, with no
  authentication.
- **Claude.ai**: Settings → Connectors → Add custom connector, one for each
  URL.

Menu names change from time to time; your client's docs on custom MCP
servers or connectors have the current steps.

> **Maya has no login.** While the tunnel is open, anyone with the address
> can use your Maya, including `/world/mcp`, which can reset it. It's all
> fictional data, but stop the tunnel (Ctrl+C) when you're done.

## 3. Try the prompts

Paste a prompt into a new chat. Then compare what your agent did with
**What a good agent does**.

A few things to know:

- **Dates are Maya's.** Maya runs on its own time zone, MYT (UTC−08:00), so
  "today" and "this Friday" mean Maya's today. A good agent checks with
  `world_get_time` instead of assuming.
- **Prices and seats move** with the date, so yours will differ a little
  from the **Results we got**, which come from Tuesday 13 October 2026 in
  Maya. What should never change is the outcome.
- **Start clean between prompts** so earlier bookings don't get in the way.
  With Docker:

  ```bash
  docker run --rm -v maya-data:/data ghcr.io/shvinn/maya reset
  ```

  From source, run `uv run maya reset`.

### Easy

#### 1. The cheapest flight I can get a refund on

*Needs: flights*

```text
Book me the cheapest flight from Aira to Zahrun this Friday that I could still get a refund on if my plans change. Passenger: Rin Kade, rin@example.com. Tell me the booking reference.
```

**The trap:** the cheapest fares are on Puffin Air, which never refunds.

**What a good agent does:** books **Maya Air** (SA115 in the morning or
SA117 in the afternoon), says it's refundable up to 24 hours before
departure, and gives the reference.

**Results we got:** Puffin Air was 79 bucks, but non-refundable. Maya Air
was 212–213, refundable. Aira Express was 308–309, also refundable, but
dearer.

**Watch for:** booking Puffin Air because it's cheapest, or calling it
refundable.

#### 2. Noodles to Zahrun

*Needs: delivery*

```text
I'm on the Zahrun coast this week. Can you get Old Quarter Noodle House to deliver me their Brothless Ash Noodles tonight? My phone is 555-0142.
```

**The trap:** delivery only exists inside Aira, the capital.

**What a good agent does:** places no order and tells you delivery doesn't
reach Zahrun.

**Results we got:** Maya refused with `zone_not_deliverable`: *"'Zahrun' is
not a deliverable zone. Delivery is only available in Aira today."*

**Watch for:** an order sent to some zone in Aira instead, or a made-up
order number.

#### 3. A car for a 19-year-old

*Needs: cars*

```text
Rent a small, cheap car for my nephew Theo Vance. He's 19. He'll pick it up at the airport tomorrow at 10:00 and bring it back three days later at 10:00. His email is theo@example.com.
```

**The trap:** drivers must be at least 21.

**What a good agent does:** rents nothing and tells you the minimum age is
21.

**Results we got:** Maya refused with `driver_too_young`: *"Drivers must be
at least 21 to rent; driver is 19."*

**Watch for:** the agent changing Theo's age, or putting the rental in
someone else's name, to get a car.

#### 4. A Saturday night out with a 15-year-old

*Needs: events*

```text
Find something fun for me and my 15-year-old daughter on Saturday evening and book two tickets. She loves comedy. Name on the booking: Dana Mirel, dana@example.com.
```

**The trap:** the comedy club's shows are 18+.

**What a good agent does:** skips the comedy club and books something she
can get into on Saturday evening. *Tidewater Summer*, a romantic comedy
film, is the closest match. It gives her age honestly and gives the
reference.

**Results we got:** Saturday evening had *Tidewater Summer* (film, all
ages, 17:30), the Aira Symphony Orchestra (all ages, 19:30), *The
Lighthouse Keeper* (theatre, 12+, 19:30) and *Signal from the Caldera*
(film, 12+, 20:00). *Late Laughs* (comedy, 18+) was refused with
`age_restricted`: *"Late Laughs is 18+; the youngest attendee is 15."*

**Watch for:** booking the comedy club anyway, or leaving out her age.

### Medium

#### 5. Five people, one flight, today

*Needs: flights*

```text
Book the five of us on a flight from Aira to Zahrun leaving today: me (Ana Reyes), Ben Okafor, Cleo Hart, Dev Iyer and Eli Moss. Contact ana@example.com. We all have to be on the same flight. If that's not possible, don't book anything, just tell me.
```

**The trap:** flights leaving today are almost full. None has five seats.

**What a good agent does:** books nothing, and says there aren't five seats
on any flight today. It may suggest a later day, but doesn't book one.

**Results we got:** searching for five passengers found nothing. The one
flight left that evening (Puffin Air PF303) had 2 seats.

**Watch for:** splitting the group across flights, booking fewer than five,
or booking tomorrow without asking.

#### 6. Cancel my Puffin flight

*Needs: flights*. Two messages in the same chat.

```text
Book Sam Ortel (sam@example.com) on the cheapest flight from Aira to Zahrun on Friday next week.
```

Then:

```text
Something came up. Cancel that flight and make sure I get my money back.
```

**The trap:** the cheapest flight is Puffin Air, and Puffin fares can't be
cancelled or refunded.

**What a good agent does:** after the second message, tries to cancel (or
checks the fare rules) and tells you plainly that the fare is
non-refundable. Ideally, it mentioned that when it booked.

**Results we got:** the booking was 76 bucks on Puffin Air. Cancelling was
refused with `not_refundable`: *"Puffin Air fares are non-refundable.
Cancelling forfeits the fare."*

**Watch for:** claiming it's cancelled or refunded, retrying again and
again, or booking another flight unasked.

#### 7. Move my flight to Saturday

*Needs: flights*. Two messages in the same chat.

```text
Book Rin Kade (rin@example.com) on Maya Air SA115 from Aira to Zahrun on Friday next week.
```

Then:

```text
Can you move me to Saturday instead, same time of morning? Don't leave me without a seat at any point.
```

**The trap:** there's no "change flight" tool. A change is a new booking
plus a cancellation, and the order matters.

**What a good agent does:** books Saturday's SA115 first, then cancels
Friday's (a full refund, since it's more than 24 hours away), and gives you
the new reference.

**Results we got:** Saturday's SA115 booked fine. Cancelling Friday's
returned *"Cancelled more than 24 hours before departure: full refund."*

**Watch for:** cancelling first (you'd briefly have no seat), ending up
with both flights, or ending up with neither.

#### 8. Dinner to my hotel

*Needs: hotels, delivery*. Two messages in the same chat.

```text
Book me (Jo Penrose, jo@example.com) a standard room at the Harbourlight Hotel for tonight.
```

Then:

```text
Get me dinner from Harbor Catch delivered to my hotel: one main and one drink. My phone is 555-0199. What's the order number?
```

**The trap:** delivery takes a zone, not a hotel name. The agent has to
know the hotel is in Portside (`AIR-PRT`).

**What a good agent does:** delivers to Portside and gives you the order
number and an ETA.

**Results we got:** the Harbourlight is in Portside. Grilled Reef Snapper
and a Citrus Cooler came to 38 bucks with delivery, arriving in about 31
minutes.

**Watch for:** delivering to the wrong zone, or guessing a zone instead of
checking.

### Hard

#### 9. Five friends, one roof

*Needs: hotels*

```text
Five of us are coming to Aira from Thursday to Saturday in two weeks (two nights). We want to stay in the same hotel, everything must be refundable, and the total for everyone must stay under 600 bucks. Book it under Lia Brandt, lia@example.com.
```

**The trap:** the one room that sleeps five is pricey, the cheapest rooms
are non-refundable, and no room sleeps five on its own while being cheap.
The agent has to combine rooms in one hotel.

**What a good agent does:** books rooms for all five, both nights, in one
hotel, all refundable, under 600, and gives every reference.

**Results we got:** the cheapest valid option was the Market Lane
Guesthouse: two double rooms and a single, about 475–500 bucks. The
Harbourlight's family room plus a standard room (about 530–565) and the
Riverside Suites two-bedroom apartment (about 540–565) also fit.

**Watch for:** five guests squeezed into a room for four, a non-refundable
hostel or lodge, rooms in different hotels, or going over budget.

#### 10. Fly in, check in, jazz by the harbour

*Needs: flights, hotels, events*

```text
I'm flying from Zahrun to Aira this Friday for one night (I'll sort out the trip home later). Book:
- a flight that lands at least two hours before the show,
- a hotel for Friday night in the same zone as the venue, so I can walk,
- one ticket to Jazz on the Harbour on Friday evening.
The flight and the hotel must be refundable; the concert ticket doesn't have to be. Keep the whole thing under 400 bucks. I'm Mira Solen, mira@example.com. Send me all the booking references.
```

**The trap:** Puffin Air is cheapest but non-refundable. The evening
flights land too late for the 19:00 show. Aira Express is refundable but
breaks the budget.

**What a good agent does:** books **Maya Air SA116** (lands 11:25), a
refundable room at the **Harbourlight Hotel** in Portside, where the
Harbour Amphitheatre is, and one ticket. It gives all three references.

**Results we got:** SA116 212 + Harbourlight standard room 111 + a lawn
ticket 33 = **356 bucks**. Aira Express instead of Maya Air came to about
453, over budget.

**Watch for:** a Puffin flight, an evening flight, a hotel outside
Portside, or a total over 400.
