# tile-rack

A simple memory cache for map tiles

Returns tiles *synchronously*. If the tile at the requested z/x/y indices
is not yet in memory, tile-rack will:
1. Return a *parent* tile, along with cropping and scaling info so you can
   stretch it to cover the area of your requested tile
2. Request the correct tile (from the supplied API), so it will be ready for
   later calls to tile-rack

## Installation
tile-rack is provided as an ESM module import.
```javascript
import { initCache } from 'tile-rack';
```

## Initialization
initCache requires two parameters:
- size: pixel size at which the (square) tiles will be displayed
- tileFactory: a function that creates and renders tiles

The supplied tileFactory function must have a method with the following
signature:
```javascript
newTile = tileFactory.create(z, x, y)
```
which will return a new tile object at the specified z, x, y indices. The
object must be returned synchronously, even though the actual image data
may not yet be ready.

The returned tile object must have the following properties:
- .rendered: A (Boolean) flag indicating whether the tile is ready to use
- .cancel(): A method to cancel any ongoing data loads or rendering tasks

The tileFactory function may optionally include a method to redraw tiles:
```javascript
tileFactory.redraw(tile);
```
If supplied, this method will be called when `tile && !tile.rendered`.

## API
Initialization returns an object with the following methods:
- `retrieve(zxy)`: Inputs an array containing the [z, x, y] coordinates of
  a requested tile, and returns either the tile, or a parent tile. Tiles are
  returned in a "box" object with sx, sy, sw properties, indicating the
  cropping and scaling required to match a parent tile to the requested
  coordinates. The tile itself can be accessed via the `box.tile` property.
- `process(func)`: Calls `func(tile)` for every tile in the cache
- `prune(metric, threshold)`: Calls `metric` method on every tile in the cache,
  and writes the returned metric value to a `tile.priority` property. Tiles 
  where `metric(tile.z, tile.x, tile.y) > threshold` are deleted from the
  cache. Return value is the number of tiles in the cache.
- `getPriority(id)`: Returns `tile.priority` for the tile with the supplied
  `id`, where `id` is constructed from the indices of the tile:
  ```javascript
  id = z + "/" + x + "/" + y`
  ```

## Use with tile-kiln
If [tile-kiln](https://github.com/jjhembd/tilekiln) is your preferred tile
factory function, you may wish to use the supplied wrapper function:
```javascript
import { wrapTileKiln } from 'tile-rack';
```
This wrapper behaves almost identically to `initCache`, except that the
returned API replaces the `process` method with the following tile-kiln
utilities:
- unrender(group): Sets .rendered flags to false for the specified tilekiln 
  layer group. Also sets the main tile.rendered flag to false, even if no
  group is specified.
- hideGroup(group), showGroup(group): Sets the .visible flag for the specified
  tilekiln layer group.
