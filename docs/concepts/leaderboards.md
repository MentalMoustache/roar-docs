## Overview

Leaderboards allow you to rank your players relative to each other for selected attributes or currencies.

#### API reference
Check out the API documentation for the two principal leaderboard calls:

- [/leaderboards/list](http://roarengine.github.io/webapi/#leaderboardslist) - Get a list of all available leaderboards
- [/leaderboards/view](http://roarengine.github.io/webapi/#leaderboardsview) - Displays the contents of a specific leaderboard


## Creating a leaderboard

Any attribute, resource or currency can be tracked as its own leaderboard.

For stats you wish to create a leaderboard for, change the `tracked` property to `true`, e.g.:

    <attribute id="3001" ikey="fame" ... tracked="true"/>

Next, you need to manually update the leaderboard DB:

1. **Add the resource**:
    - `app_id`: Your app ID. See _apps_ table. Default: 3047
    - `resource_id`: The unique ID for your resource
    - `resource_ikey`: Unique ikey for your resource
    - `current_version`: Set to 1
2. **Add the board**:
    - `board_id`: ID for this leaderboard (rec. match resource ID)
    - `app_id`: Your app ID
    - `resource_id`: The ID of the resource you're tracking
3. **Add permissions**:
    - `user_id`: The ID of your leaderboard user. See _users_ table.
    - `app_id`: Your app ID
    - `resource_id`: The ID of the resource you're tracking
    - `permission`: Set to "ALL"

**Note**: Leaderboards track changes, they are not historical. If you create a leaderboard for a stat after your game has launched, it will only track player stats from that point on. This means that an old player with a very high score will not be added into the leaderboard until they log into your game and update that stat in some way.


## Returning Extra Display Data

The default data that gets returned with for a player entry for given leaderboard is:

`<entry rank="21" player_id="12345..." value="500"/>`

There are two ways to provide additional information.

#### 1. Compound leaderboard data
If you wish to include additional information on other stats along with the tracked leaderboard, you can update the `compound` property on a stat:

    <attribute id="3001" ikey="fame" ... compound="true"/>

This will return the `fame` value for all returned players on every leaderboard call.

For example, if you were tracking leaderboards for `mojo`, but also wanted to display the ranked players `fame` and `game_coins` in your UI, you could display the leaderboard for `mojo` and return the `fame` and `game_coins` values for those players by setting the **compound** value to `true` in the config.

The resulting entry would look like this:

    <entry rank="21" player_id="12345..." value="500">
        <custom>
            <stat ikey="fame" value="30"/>
            <stat ikey="game_coins" value="125"/>
        </custom>
    </entry>

**Note:** A stat *does not* need to be tracked as a leaderboard to be returned as a compound field.

#### 2. Custom properties
You can optionally configure a Custom Property to return as data for a player in **all** leaderboards. This is particularly useful to return client-configurable information such as "Character Name" or "Band Label".

To enable this functionality, set the `compound` value to `true`.

    <attributes ikey="hero_name" ... compound="true"/>

For example, if you create a `hero_name` custom property, and set the `compound="true"`, this value will be represented as part of a leaderboard entry as follows:

    <entry rank="21" player_id="12345..." value="500">
        <custom>
            <property ikey="hero_name" value="Mashton Thunderwalker"/>
        </custom>
    </entry>



## Listing and Finding
It is possible to query Roar for a list of all the stats you have made available as leaderboards. Using this information your client can then request the details of a specific board. However you can optionally hardcode a lookup for a specific leaderboard.

#### API call
**[/leaderboards/list](http://roarengine.github.io/webapi/#leaderboardslist)**
Shows a list of all available leaderboards (these are configured in the web dashboard to track any attributes, resources or currencies you choose).

#### Inputs
None

#### Returns:
  <roar tick="0">
   <leaderboards>
      <list status="ok">
          <board board_id="4001" ikey="premium_currency" label="Super Coins" board_label="Most Super Coins"/>
          <board board_id="4002" ikey="mojo" label="Mojo" board_label="Highest Mojo"/>
          <board board_id="4003" ikey="notoriety" label="Notoriety" board_label="Most Notorious"/>
      </list>
   </leaderboards>
  </roar>


## Displaying

The default leaderboard request will return the a) first, b) 100 players, c) within the entire game, who have the d) highest values for that given stat. It is possible to adjust all of these parameters to configure:

a. Returning the starting position of the rankings  (using **page** or **offset** depending on your needs)
b. The number of results (using **num_results**)
c. The list of players to select from (using **ids** to override the *global* default)
d. Whether low scores should rank at the top (setting **low_is_high** to true)

#### Displaying a custom list of players (such as friends)
You can build a custom list of ranked players by providing their player_ids to the **ids** input parameter for the API call. If the **ids** parameter is set, **num_results**, **page** and **offset** are ignored. To get a list of a player's friends use the [/friends/list/](http://roarengine.github.io/webapi/#friendslist) API call.

**Note**: You can pass in a maximum of 250 ids this way.


#### API call
**[/leaderboards/view](http://roarengine.github.io/webapi/#leaderboardsview)**
Displays a specific leaderboard

#### Inputs

- **board_id:** The unique key for the leaderboard you wish to show
- **num_results:** *(optional - Default 100)* How many results to return (length of the board, eg. Top 5, 25, 100, etc)
- **offset:** *(optional - Default 0)* Where to 'start' the list. If offset is 25.
- **low_is_high:** *(optional - Default false)* Sets the sort direction (forcing this value to true means low scores are best)
- **page:** *(optional - Default 1)* Pagination id. eg. If you wish to view results 101-200, you'd leave all defaults and set page to 2.

- **ids:** *(optional - Default null)* Array of player_ids, comma separated with no spaces, to build the leaderboard from (can be a list of friends, enemies, guild, etc)

- **player_id:** *(optional)* A specific player to return with their rank, regardless if they appear within the requested set of players

#### Example

API call: **/leaderboards/view/**
POST variables:

- board_id: 4002
- num_results: 1
- compound: [ game_coins ]
- player_id: 125592045613

#### Returns
  <roar tick="0">
   <leaderboards>
      <view status="ok">
                <ranking key="mojo" offset="0" num_results="100" page="1" low_is_high="false">
                    <entry rank="1" player_id="52992481850" value="600">
                        <custom>
                            <stat ikey="game_coins" value="350"/>
                            <property ikey="hero_name" value="Mashton Thunderwalker"/>
                        </custom>
                    </entry>
                    <!-- ...etc... -->
                </ranking>

                <!-- Response also includes requesting player's ranking as a distinct entry -->
                <me>
                    <entry rank="75" player_id="125592045613" value="129">
                    </entry>
                </me>
      </view>
   </leaderboards>
  </roar>


