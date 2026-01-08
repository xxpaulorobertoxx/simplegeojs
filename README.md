# geojs
Simple geo lib for Next.js and GitHub Pages. It stores items with latitude/longitude,
supports add/remove, and performs radius search using math only (Haversine distance).

## Features
- ESM-first, no dependencies.
- Immutable operations (returns new arrays).
- Works in Next.js and static builds (GitHub Pages).
- Runtime validation for coordinates, id, and data.

## Installation
```bash
npm install github:xxpaulorobertoxx/geojs
```

## Basic usage
```js
import { addItem, removeItemById, findWithinRadius } from "geojs";

const items = [];

// addItem returns a new array with the item appended
const withItem = addItem(items, {
  id: "poi-1",
  latitude: -23.55052,
  longitude: -46.633308,
  data: { name: "Sao Paulo", kind: "city" }
});

// findWithinRadius returns only items inside the given radius in meters
const nearby = findWithinRadius(withItem, -23.55052, -46.633308, 1000);
// removeItemById removes items with the provided id
const cleaned = removeItemById(withItem, "poi-1");
```

## Next.js usage
This library is pure JavaScript and safe for both server and client components.
Example in a client component:

```js
"use client";

import { addItem, findWithinRadius } from "geojs";
import { useMemo, useState } from "react";

export default function MapPanel() {
  const [items, setItems] = useState([]);

  function addSample() {
    setItems((current) =>
      // addItem returns a new array, so it's safe for React state updates
      addItem(current, {
        id: Date.now(),
        latitude: 40.7128,
        longitude: -74.006,
        data: { label: "NYC" }
      })
    );
  }

  const nearby = useMemo(
    // findWithinRadius filters points by distance using math only
    () => findWithinRadius(items, 40.7128, -74.006, 2000),
    [items]
  );

  return (
    <div>
      <button onClick={addSample}>Add</button>
      <div>Nearby: {nearby.length}</div>
    </div>
  );
}
```

## Data model
Each item must include:
- `id`: string or number (non-empty).
- `latitude`: number between -90 and 90.
- `longitude`: number between -180 and 180.
- `data`: object (any shape, used to store custom metadata).

Example:
```js
{
  id: "place-123",
  latitude: 51.5072,
  longitude: -0.1276,
  data: { label: "London", category: "city" }
}
```

## API

### addItem(items, item)
Adds an item to the array and returns a new array.

```js
const next = addItem(items, item);
```

### removeItemById(items, id)
Removes all items that match the given id and returns a new array.

```js
const next = removeItemById(items, "poi-1");
```

### findWithinRadius(items, latitude, longitude, radiusMeters)
Filters items within `radiusMeters` of the provided center point.
Uses the Haversine formula (spherical Earth approximation).

```js
const within2km = findWithinRadius(items, 48.8566, 2.3522, 2000);
```

### distanceMeters(lat1, lon1, lat2, lon2)
Returns the distance in meters between two points.

```js
const meters = distanceMeters(0, 0, 0, 1);
```

## Validation behavior
The functions throw errors when inputs are invalid:
- `TypeError` for non-array `items`, missing/invalid `id`, non-object `data`.
- `RangeError` for latitude/longitude out of range.
- `RangeError` if `radiusMeters` is negative.

## Notes on precision
Haversine is accurate enough for most app use cases. For very large radii or
high-precision geodesy, consider a more specialized library.

## Testing
```bash
npm test
```

## Publishing
```bash
npm publish --access public
```
