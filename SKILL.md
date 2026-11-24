---
name: travel-buddy
description: Turns your trip description into an interactive journey map and a timeline spreadsheet. Use when someone describes a holiday, route, or multi-stop trip and wants an itinerary, a map, or a file they can open in a browser.
---

# Travel Buddy

Turn a trip described in ordinary language into two things the traveller can check:

1. A day-by-day itinerary in Markdown.
2. One HTML file, copied from `template/journey-map.html`, with the trip filled into the `TRIP` object.

The map is a single file. It needs no build. It shows curved legs, a scrollable itinerary (a bottom sheet on a narrow screen), filters, a view switcher, a Play journey tour, and a popup for each place with links to explore further.

Read `template/journey-map.html` before you generate a map. The field guide sits in the comment above `TRIP`. Replace that object and leave the rest of the file byte for byte.

## How a session goes

### 1. Read the description

Note what is already decided: dates, who is travelling, the order of stops, nights in each place, how they travel between stops, pace, must-sees, and anything already booked.

### 2. Ask one short batch, then stop

Ask only about gaps that would change the itinerary or the map. Skip anything the text already answers. Put the questions in one message, grouped, not as an interrogation. Five or six is plenty. If only one or two things are missing, ask those. If nothing important is missing, say what you are assuming in a sentence and go straight to the draft.

Useful things to resolve, when they are actually unknown:

- Exact dates, including the year, and which days are travel days.
- Whose names should appear on the map, if any.
- Where the trip starts and where it ends. The home airport is usually a gateway stop.
- The order of stops, and how many nights in each.
- The mode of transport on every hop, including a change of trains or a connecting flight.
- Pace, and any days that should stay empty.
- Must-sees, and anything they want left off the map.
- What is already booked (rooms, transport, tickets), and whether those booking references should be printed.
- Optional ideas, as distinct from the plan.
- A highlight moment, if they have one (a meal, a wedding, a match).

Do not ask about budget, seat numbers, or packing unless the answer would change the plan. Do not invent booking references, prices, emails, phone numbers, or street addresses. If a reference is useful and they have not given one, leave `refs` empty.

### 3. Draft the itinerary and wait

Write a Markdown itinerary they can correct:

- A title, the dates, who is travelling, and the pace.
- One section per day: the date, a short title, then the moves and the stays.
- Mark anything optional as optional.
- End by asking them to confirm or change it.

Wait for that confirmation before you write the HTML, unless they have already told you to go ahead and build the map.

### 4. Fill the template

1. Copy `template/journey-map.html` to a new file named for the trip.
2. Replace the `TRIP` object, and only that object.
3. Keep the JSON valid: double quotes, no trailing commas, no comments inside the object.
4. Do not put the characters `</script>` in any string.

Then check, before you hand the file over:

- Every `segments.from` and `segments.to` is a stop id.
- Every day entry id exists.
- Every `coords` pair is `[latitude, longitude]`, in that order.
- `gateway: true` is only on the home end of the trip (the place that should disappear on "The trip" view).
- Optional hops use `"optional": true` and are not also listed inside a day. The engine files them under Optional ideas, and Play journey skips them.
- A connecting flight or a train with a change is one leg with several segments, in order.
- Each stop has at least a map link when you know the place. Add Wikipedia and an official site when a real public page exists.
- Labels inside one popup are distinct. Two links both called Google Maps are not allowed. Name the place, for example `Map: Tejeda`, `Wikipedia`, `Official site`.
- Day ids and leg ids must not look alike. Use `d1` for a day and `f1` or `t1` for a leg. A day called `3` and a leg called `3` get mixed up by the day filter and the tour.
- `cartoApiKey` stays `""` unless they gave you a key.
- Booking references go in `refs` (a string or a list), not in the prose. The map shows them as chips. Do not invent any.
- A day trip out and back from the same base is one leg with `"returns": true`, not two heavy arcs on top of each other.
- Idea rows that need a place line set `subtitle` (for example `Near Meloneras`).
- Chip wording is `modeStyles`. To label the road chip Drives, set `"drive": { "label": "Drives", "name": "Drive" }`.

Tell them how to open the file (double-click it, or drag it into a browser) and how to add a CARTO key if they want Voyager tiles. With no key the map uses Esri World Topo, then Esri World Street if those tiles fail. The legend names that basemap. Voyager hides the note. Export, in the top bar, downloads a CSV of the same trip. You can also write that CSV yourself, next to the HTML, using the columns below.

## Filling TRIP

A minimal shape:

```json
{
  "title": "Trip title",
  "subtitle": "Names, or leave empty",
  "dates": "7 to 20 Jun 2027",
  "footer": "Times are local.",
  "doneMessage": "Journey complete.",
  "cartoApiKey": "",
  "countdown": { "start": "2027-06-07T09:10:00+01:00", "end": "2027-06-20T20:00:00+01:00" },
  "defaultView": "region",
  "views": [
    { "id": "region", "label": "The trip" },
    { "id": "full", "label": "Full route" }
  ],
  "modes": ["flight", "ferry", "train", "drive", "idea"],
  "modeStyles": {},
  "categories": [
    { "id": "stay", "label": "Stays" },
    { "id": "sight", "label": "Sights" }
  ],
  "highlight": null,
  "stops": [],
  "legs": [],
  "days": []
}
```

Omit `countdown` to hide the "days to go" line. Set `highlight` to `{ "title", "when", "stopId", "note" }` for one pinned moment, or `null`.

### Stops

`id` (short, stable), `n` (the badge: `1`, `2`, or `A` for an idea), `name`, `coords`, `status`, `category`, `gateway`, `when`, `zoom`, `lines` (short sentences), `note`, `refs`, `links`, `optional`.

`status` is one of:

- `booked` for a ticketed stop
- `base` for a booked stay (filled marker)
- `travel` for an airport, station, or port
- `planned` for the rest of the plan
- `idea` for an optional place (`optional: true` as well)

`gateway: true` hides that stop, and any leg that touches it, on every view except `full`. Use it for the city they depart from and return to, not for every airport on the route.

`zoom` is the level Play and a click fly to. About 8 or 9 for an airport, 12 or 13 for a town.

`links` are `{ "label", "url" }` and must be `http` or `https`. Each label in that popup must be different. Use `Map: Tejeda`, not a second `Google Maps`. Wikipedia and Official site are the usual other labels. A map search URL looks like `https://www.google.com/maps/search/?api=1&query=Bryggen+Bergen`.

`refs` is one booking reference, or a list of them. The map prints each one as a chip in the popup and in the sidebar. Leave it empty when the traveller did not give you a reference.

`subtitle` is an optional second line. On an idea, that is where "Near Meloneras" belongs.

`essentials` is an optional list pinned above the days: the base hotel, the car, the tickets they must not lose. Each item is `{ "id", "title", "text", "refs", "stopId", "links", "status" }`. `stopId` is the pin a click should open. Links sit inside the card. The block starts collapsed so day 1 stays in view. Omit `essentials` when there is nothing to pin. Those rows are included at the top of the spreadsheet export, with type `essential`.

A stop with `status` `travel` (a hub or a gateway) exports as booked when it has `refs`. Set `status` to `planned` if a reference should not count as booked.

Use coordinates you are sure of for public places. If a pin is only approximate, say so in `note`.

### Legs

`id`, `mode`, `day` (a day id, or `null`), `optional`, `status`, `title`, `when`, `detail`, `links`, `segments`.

`status` on a leg is `booked`, `planned`, or `optional`. The spreadsheet export reads it. If you leave it off, a leg with `refs` exports as booked, an optional leg as optional, and every other leg as planned.

Do not give a leg the same id as a day. `d1` is a day, `f1` is a leg. Ids that look alike make the day filter and Play journey open the wrong row.

`returns: true` draws one faint arc from the end of that leg back to its start. It does not reverse every segment. `returnTo` is a stop id when the base is not the start of the leg: one faint arc from the end of the leg back to that stop.

For a hub day with several stops (hotel, then two sights), set `returnTo` on the day instead. The faint arc runs from the day's last stop back to that base, once. A one-way travel day leaves both off. Set `returnArcs` to `"none"` on the trip if you do not want faint arcs at all. The default is `"faint"`.

`start` and `end` are clock times (`09:10`, `12:05`) used by the spreadsheet export. If you leave them off, the map tries to read a `HH:MM to HH:MM` pair out of `when`.

`mode` is `flight`, `ferry`, `train`, `drive`, `bus`, `walk`, or `idea`. The engine already styles those. `modes` on TRIP lists which chips to show. Include `idea` if anything is optional.

`segments` is a list of `{ "from", "to", "c" }`. `c` bends the line in Mercator space: positive bows to the left of travel, negative to the right. Use about `0.15` on a long flight and `0.25` to `0.45` on a short hop. Give a return over the same pair the opposite sign so the two lines do not sit on top of each other.

To add a mode the engine does not know, put it in `modeStyles` instead of editing the script:

```json
"cablecar": { "label": "Cable car", "name": "Cable car", "color": "#7c3aed", "dash": "4 4", "weight": 2.6, "icon": "walk" }
```

`icon` must be one of `plane`, `ferry`, `train`, `car`, `bus`, `walk`, `idea`.

### Days

In travel order. Each day is `{ "id", "date", "title", "returnTo", "entries" }`. `returnTo` is the base stop id for one faint arc home from the day's last stop.

An entry is one of:

- `{ "type": "leg", "id": "f1" }`
- `{ "type": "stop", "id": "bgo" }`
- `{ "type": "note", "title": "Nothing planned", "text": "A slow morning.", "stopId": "bgo" }`

Use a note for a day with no transfer. `stopId` is the place a click should open. Do not put optional legs or idea stops in a day. They are collected automatically.

`day` on a leg must match the day that leg appears on, or Play journey and the day filter will disagree. The id of the day and the id of the leg still have to be different strings.

## What the map already does

You do not write any of this. It comes from the template.

- Curves and arrowheads are computed in Web Mercator, so they match the tiles at every zoom. Colour and dash follow the mode: flights dashed, ferries long-dashed, trains and roads solid, optional ideas dotted.
- The itinerary lists the days in order. On a phone it is a bottom sheet: a peek of the title, drag or tap the handle for the full list. The open row and the map stay in sync.
- Chips filter by transport. The two menus filter by day and by place category. Show all puts them back.
- "The trip" hides gateway stops. "Full route" brings them, and the long legs home, back into frame.
- Play journey draws each planned leg in order, with a moving marker. Space starts and pauses. Arrow keys step. Escape closes the tour and returns the map to the view that was active before Play (The trip, or Island, not whatever the last leg zoomed to). Optional ideas are not part of the tour.
- A click on a place opens a popup with the notes and the explore links. A click on a line opens the leg.
- Tiles: if `cartoApiKey` is set, or the page was opened with `?cartoKey=YOUR_KEY` (stored in that browser), the basemap is CARTO Voyager and the legend note stays hidden. With no key the map uses Esri World Topo, then Esri World Street if that layer fails. OpenStreetMap is only tried when the page itself is served over http or https. The legend names the basemap in use and mentions how to add a CARTO key. Attribution stays on the map. The legend starts collapsed so it does not cover the route.
- Export downloads `<trip-slug>-timeline.csv` (UTF-8 with a BOM, so Excel keeps accents). Copy puts the same table on the clipboard as TSV for Google Sheets. Link columns are `=HYPERLINK("url","label")` formulas, and `All links` repeats the raw URLs. Text that starts with `=`, `+`, `-`, or `@` is prefixed with an apostrophe so it cannot become a formula. You can write this file yourself beside the HTML. Columns, in order: Date, Day label, Start time, End time, Type (essential, flight, ferry, train, drive, bus, walk, stop, activity, or idea), Title, From, To, Place, Transport mode, Status (booked, planned, or optional), Refs, Notes, Map link, Explore link 1 to n, All links. Essentials are the first rows.

## Worked example

`examples/example-prompt.md` is a fictional brief. `examples/example-itinerary.md` is the draft after one round of answers. `examples/north-sea-fortnight.html` is the map those answers produced. The people, dates, and booking references in that example are invented.
