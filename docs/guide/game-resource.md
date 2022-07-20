
# Game resource

### Description

With the Game API, you can obtain the fantasy game related information, like the fantasy game name, the Yahoo! game code, and season.

To refer to a Game resource, you'll need to provide a `game_key`, which will either be a `game_id` or game_code. The game_id is a unique ID identifying a given fantasy game for a given season. For instance, the `game_id` for the Free NFL draft and trade fantasy game for the 2009 season is 222, while the `game_id` for the Plus version is 223. A `game_code` generally identifies a game, independent of season, and, when used as a `game_key`, will typically return the current season of that game. For instance, the `game_code` for the Free NFL game is nfl, and the `game_code` for the Plus game is pnfl; using nfl as your game_key during the 2010 season would be the same as providing the `game_id` for the 2010 season of the NFL game (242). As of the 2010 seasons, the Plus and Free games have been combined into a single code. Next year, the `game_code` nfl will point to the new `game_id` for the 2011 version of the NFL game. Thus, if you always want the current season of a game, the `game_code` should be used as a `game_key`.

Below is a list of game IDs for most of our seasons of each game. If you're looking for a current game ID that's not listed in the table below, as mentioned above, you can request game information by `game_code`. For example:

    YQL: select * from fantasysports.games where game_key='nfl';
    API: http://fantasysports.yahooapis.com/fantasy/v2/game/nfl

### Game IDs Table

| Season | nfl game ID | pnfl game ID |	mlb game ID | pmlb game ID | nba game ID | nhl game ID |
|--------|-------------|--------------|-------------|--------------|-------------|-------------|
| 2001 |  57  | 58 | 12  | --  |	16  | 15 |
| 2002 |  49  | 62 | 39  | 44  |	67  | 64 |
| 2003 |	79  | 78 | 74  | 73  |	95  | 94 |
| 2004 |	101 | 102| 98  |	99  |  112 | 111 |
| 2005 |	124 | 125| 113 |	114 |	131 | 130 |
| 2006 |	153 | 154| 147 |	148 |	165 | 164 |
| 2007 |	175 | 176| 171 |	172 |	187 | 186 |
| 2008 |	199 | 200| 195 |	196 |	211 | 210 |
| 2009 |	222 | 223| 215 |	216 |	234 | 233 |
| 2010 |	242 | -- | 238 |	--  | 	249 | 248 |
| 2011 |	257 | -- | 253 |	--  |	265 | 263 |
| 2012 |	273 | -- | 268 |	--  |	304 | 303 |

### HTTP Operations Supported

	GET

### URIs

	http://fantasysports.yahooapis.com/fantasy/v2/game/{game_key}

Any sub-resource under a game is extracted using a URI like:

	http://fantasysports.yahooapis.com/fantasy/v2/game/{game_key}/{sub_resource}

Multiple sub-resources can be extracted from game in the same URI using a format like:

	http://fantasysports.yahooapis.com/fantasy/v2/game/{game_key};out={sub_resource_1},{sub_resource_2}

### Game key format

	{game_code} or {game_id}

	Example: pnfl or 223

### Note

If you specify a `game_code` as the `game_key`, we'll translate that to the corresponding game_id upon parsing the URI. Therefore, any `game_codes` will be converted to `game_ids` in any keys returned by the Fantasy Sports APIs in the response XML.

### Sub-resources

Default sub-resource: metadata

| Name         | Description | URI        | Sample |
|--------------|-------------|------------|--------|
| metadata | Includes game key, code, name, url, type and season.  | /fantasy/v2/game/{game_key}/metadata  | The 2009 Football PLUS game: http://fantasysports.yahooapis.com/fantasy/v2/game/223 |
|leagues | 	Fetch specified leagues under a game.  |fantasy/v2/game/{game_key}/leagues;league_keys={league_key1},{league_key2}  | A publicly viewable league within the 2009 football plus game: http://fantasysports.yahooapis.com/fantasy/v2/game/223/leagues;league_keys=223.l.431 |
| players |Fetch specified players under a game.  |/fantasy/v2/game/{game_key}/players;player_keys={player_key1},{player_key2}  |Brett Favre's information from the 2009 football plus game: http://fantasysports.yahooapis.com/fantasy/v2/game/223/players;player_keys=223.p.1025 |
| game_weeks | Start and end date information for each week in the game|/fantasy/v2/game/{game_key}/game_weeks |NFL game weeks: http://fantasysports.yahooapis.com/fantasy/v2/game/nfl/game_weeks |
|stat_categories | Detailed description of all available stat categories for the game.  |/fantasy/v2/game/{game_key}/stat_categories  |NFL stat categories: http://fantasysports.yahooapis.com/fantasy/v2/game/nfl/stat_categories |
| position_types|Detailed description of all player position types for the game. |/fantasy/v2/game/{game_key}/position_types | NFL position types: http://fantasysports.yahooapis.com/fantasy/v2/game/nfl/position_types |
| roster_positions |Detailed description of all roster positions for the game. |/fantasy/v2/game/{game_key}/roster_positions  | NFL roster positions: http://fantasysports.yahooapis.com/fantasy/v2/game/nfl/roster_positions |

### Sample XML

http://fantasysports.yahooapis.com/fantasy/v2/game/nfl

```xml
<?xml version="1.0" encoding="UTF-8"?>  
    <fantasy_content xml:lang="en-US" yahoo:uri="http://fantasysports.yahooapis.com/fantasy/v2/game/nfl" xmlns:yahoo="http://www.yahooapis.com/v1/base.rng" time="30.575037002563ms" copyright="Data provided by Yahoo! and STATS, LLC" xmlns="http://fantasysports.yahooapis.com/fantasy/v2/base.rng">  
      <game>  
        <game_key>257</game_key>  
        <game_id>257</game_id>  
        <name>Football</name>  
        <code>nfl</code>  
        <type>full</type>  
        <url>http://football.fantasysports.yahoo.com/f1</url>  
        <season>2011</season>  
      </game>  
    </fantasy_content>  
```