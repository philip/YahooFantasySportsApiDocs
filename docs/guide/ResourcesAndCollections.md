
# Resources and Collections

### Introduction

The primary building blocks of the Fantasy Sports APIs are Resources and Collections. Resources typically describe chunks of data that can be identified by a unique key. Collections are simply wrappers that contain similar resources. So, for instance, if we need to retrieve data about a single league, we might ask for a League Resource and provide a single league key. However, if we wanted data across several leagues, we would ask for a Leagues Collection and provide multiple league keys.

The format for requesting a Resource will typically look like:

	`http://fantasysports.yahooapis.com/fantasy/v2/{resource}/{resource_key}`

While the format for requesting a Collection will typically look like:

	`http://fantasysports.yahooapis.com/fantasy/v2/{collection};{resource}_keys={resource_key1},{resource_key2}`

### Collections

As mentioned, Collections are simply groups of Resources. If you care about particular Resources within a Collection, you can apply filters to the Collection to narrow the results. The most common type of filtering is by key. For instance, if you'd like to see two particular players, you could ask for them directly:

	`/fantasy/v2/players;player_keys={player_key1},{player_key2}`

Some Collections support more complex filters. Within a game, for example, you might ask for only the players that play a certain position. You could also request only the particular user who is currently logged in:

	`/fantasy/v2/users;use_login=1`

### Sub-Resources

Resources will typically define a list of valid Sub-Resources. These are Resources and Collections that can live within the scope of the parent Resource. For instance, a fantasy league in the Football draft and trade games can contain up to 20 fantasy teams; therefore, the League Resource can have a Teams Collection as a sub-resource. As a general rule of thumb, if you can possibly have multiple of one Resource contained within another Resource, then you'll have a Collection of the first Resource as a sub-resource of the second Resource. You would only have a singular Resource as a sub-resource of another Resource if it would only make sense to ever ask for one instance of that Resource.

The scope of a sub-resource is typically defined by the parent Resource; for instance, if you're viewing a Players Collection as a sub-resource of a particular League, then you would expect to only see Players that are eligible within that League. Further filters could then be applied to this already narrowed list.

Having sub-resources allows you to chain together Resources and Collections to provide more data, and the URI you request directly specifies how the chaining works. For instance, if you wanted to take a particular logged in user, see which games he had played, and then get the league information within those games, you might construct a request like:

	`/fantasy/v2/users;use_login=1/games/leagues`

This would present you with a Users Collection, a single User Resource for the logged in user, a Games Collection for that user, potentially multiple Game Resources for each game the user is playing, a Leagues Collection beneath each Game Resource, and potentially multiple League Resources for each league the user belongs to in that game.

When you specify a sub-resource beneath a Collection, you're really saying that you'd like to see that sub-resource appended beneath each Resource within the Collection. Therefore, the sub-resources available to a Collection will be equivalent to the sub-resources available to the corresponding Resource.

If you ever need to branch off other sub-resources outside of your main resource chain, you can use the out parameter, which will let you specify one level of extra sub-resources to pull in. At the moment, you cannot pass any parameters along to these out sub-resources, aside from any data that might get passed by default. This typically means that you can't chain other resources off of sub-resources specified by the out parameter.

As an example, if you wanted to view a league's settings along with two teams in particular in a league, you might construct a URI like:

	`/fantasy/v2/league/{league_key};out=settings/teams;team_keys={team_key1},{team_key2}`

### Parameters

Parameters can be provided to Resources and Collections as semicolon-delimited key-value pairs. These should be placed after the Resource or Collection name in the URI; in the case of entry-point Resources like Games, Leagues, Teams, and Players, the parameters belong after the resource_key.

	`/fantasy/v2/{resource}/{resource_key};{key}={value};{key}={value}/{collection};{key}={value};{key}={value}/{collection};{key}={value}/{resource};{key}={value}`

Resource keys, out parameters, and other filters are just specific types of parameters that can be applied to various Resources or Collections.
