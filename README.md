# maya-agent-scenarios

Scenarios for testing AI agents in [Maya](https://github.com/shvinn/maya), a
simulated world of everyday services (flights, hotels, car rental, food
delivery and events) served as MCP tools.

Each scenario is a task an agent has to carry out in Maya, with a trap that
a real service would set: the cheapest fare can't be refunded, there aren't
enough seats, the driver is too young, the user gives a booking reference
instead of an address. Some can only be passed by saying no, honestly.

Maya knows nothing about these scenarios. They use Maya only through its MCP
tools, like any other client.

> **Status: early drafts.** The ten scenarios in [`scenarios/`](scenarios/)
> are a starting point. Their tasks and traps are settled, but the check
> format will change, and there's no runner yet.

## How a scenario runs

1. **Reset Maya** with `world_reset`, and move the clock to the scenario's
   start with `world_set_time`. Both are tools on Maya's `world` server.
2. **Set up** anything the scenario needs beforehand, such as an existing
   booking, with Maya's domain tools.
3. **Give the agent the task.** It works through Maya's domain servers (not
   the `world` server), then writes a report of what it did.
4. **Grade the run** against the report and against Maya's final state,
   read through the domain list tools (`flights_list_bookings` and others).

## The scenarios

| Scenario | Level | Domains | The trap |
| --- | --- | --- | --- |
| `flight-refundable` | easy | flights | The cheapest fares are on Puffin Air, which never refunds |
| `delivery-out-of-town` | easy | delivery | Delivery only reaches Aira; the user is in Zahrun |
| `car-young-driver` | easy | cars | Drivers must be 21; the nephew is 19 |
| `events-with-a-teen` | easy | events | Every comedy club show is 18+; the daughter is 15 |
| `flight-change` | medium | flights | There's no change tool: book the new flight, then cancel the old one |
| `flight-not-refundable` | medium | flights | A Puffin Air fare can't be cancelled or refunded |
| `flight-sold-out` | medium | flights | Three travellers, and every flight today has two seats |
| `dinner-to-my-hotel` | medium | hotels, delivery | The address is a hotel booking reference; its zone has to be looked up |
| `hotel-group-of-five` | hard | hotels | No cheap, refundable room sleeps five: combine rooms in one hotel, under budget |
| `trip-to-aira` | hard | flights, hotels, events | Flight, hotel and a show on one budget; the cheap flight isn't refundable, the evening ones land too late |

The scenarios are set in March 2030, so their start time is always in the
future: Maya's clock only moves forward.

## License

MIT. See [LICENSE](LICENSE).
