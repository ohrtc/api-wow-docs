# WoW Community Web API

This is the documentation for the RESTful APIs exposed through the World of Warcraft community site as a service to the World of Warcraft community.

## Introduction

The Blizzard Community Platform API provides a number of resources for
developers and Wow enthusiasts to gather data about their characters,
guilds and arena teams. This documentation is primarily for developers
and third parties.

Blizzard's epic gaming experiences often take place in game, but can
lead to rewarding and lasting experiences out of game as well. Through
exposing key sets of data, we can enable the community to create
extended communities to continue that epic experience.

## Features

Before getting started with the Community Platform API, programmers
must first understand how the API is organized and how it works. The
following sections provide a high level overview of the features of
this API. It is recommended that the reader have knowledge of the HTTP
protocol as well as a general understanding of web technologies.

### REST

The API is mostly RESTful. Data is exposed in the form of URIs that
represent resources and can be fetched with HTTP clients (like web
browsers). At this time, the API is limited to read-only operations.

The character and guild API resources do honor HTTP requests that
contain the "If-Modified-Since" header.

### Access and Regions

To access the API, HTTP requests can be made to specific URLs and
resources exposed on the regional Battle.net domains.

*An example API request and response.*
```plain
GET /api/wow/realm/status HTTP/1.1
Host: us.battle.net
<http headers>
```
```plain
HTTP/1.1 200 OK
<http headers>

{"realms":[ ... ]}
```

The regions where this API is available are:

* us.battle.net
* eu.battle.net
* kr.battle.net
* tw.battle.net
* www.battlenet.com.cn

The data available through the API is limited to the region that it is
in. Hence, US APIs accessed through us.battle.net will only contain
data within US battlegroups and realms. Support for locales is limited
to those supported on the World of Warcraft community sites.

#### Localization

All of the API resources provided adhere to the practice of providing
localized strings using the locale query string parameter. The locales
supported vary from region to region and align with those supported on
the community sites.

* us.battle.net
  * en_US
  * es_MX
  * pt_BR
* eu.battle.net
  * en_GB
  * es_ES
  * fr_FR
  * ru_RU
  * de_DE
  * pt_PT
* kr.battle.net
  * ko_KR
* tw.battle.net
  * zh_TW
* www.battlenet.com.cn
  * zh_CN

### Throttling

Consumers of the API can make a limited number of requests per day, as
stated in the API Policy and Terms of Use. For anonymous consumers of
the API, the number of requests that can be made per day is set to
3,000. Once that threshold is reached, depending on site activity and
performance, subsequent requests may be denied. High limits are
available to registered applications.

### Authentication

Although most of the application can be accessed without any form of
authentication, we do support a form application registration and
authentication. Application authentication involves creating and
including an application identifier and a request signature and
including those values with the request headers.

The primary benefit to making requests as an authenticated application
is that you can make more requests per day.

#### Application Registration

To send authenticated request you first need to register an
application. Because registration isn't automated, application
registration is limited to those who meet the following criteria:

* You plan on making requests from one or more IP addresses. (e.g. a
  production environment and development environment)

* You can justify making more than 2,000 requests per day from one or
  more IP addresses.

Registering an application is a matter of providing a description of
the application, how you plan on using the API and your contact
information to
[api-support@blizzard.com](mailto:api-support@blizzard.com) with the
subject "Application Registration Request". Once we receive your
request, we will contact you to either provide additional information
or with application keys to use.

#### Authentication Process

To authenticate a request, simple include the "Authorization" header
with your application identifier and the request signature.

*An example authenticated request*
```plain
GET /api/wow/character/Medivh/Thrall HTTP/1.1
Host: us.battle.net
Date: Fri, 10 Jun 2011 20:59:24 GMT
Authorization: BNET c1fbf21b79c03191d:+3fE0RaKc+PqxN0gi8va5GQC35A=
```

In the above exmple, the value of the Authorization header has three
parts `"BNET"`, `"c1fbf21b79c03191d"` and
`"+3fE0RaKc+PqxN0gi8va5GQC35A="`. The first part is a processing
directive for the Authorization header. The second and third values
are the application public key and the request signature. The
application public key is assigned by Blizzard during the application
registration process. The signature is generated with each request and
is discribed by the following algorithm.

```plain
UrlPath = <HTTP-Request-URI, from the port to the query string>

StringToSign = HTTP-Verb + "\n" +
    Date + "\n" +
    UrlPath + "\n";

Signature = Base64( HMAC-SHA1( UTF-8-Encoding-Of( PrivateKey ), StringToSign ) );

Header = "Authorization: BNET" + " " + PublicKey + ":" + Signature;
```

The above process can be seen in action by filling in the blanks:

```plain
UrlPath = "/api/wow/realm/status"

StringToSign = "GET" + "\n" +
    "Fri, 10 Jun 2011 21:37:34 GMT" + "\n" +
    UrlPath + "\n";

Signature = Base64( HMAC-SHA1( UTF-8-Encoding-Of( "examplesecret" ), StringToSign ) );

Header = "Authorization: BNET" + " " + "examplekey" + ":" + Signature;
```

The date timestamp used in the above algorithm and example is the
value of the Date HTTP header. The two date values, the first being
used to sign the request and the second as sent with the request
headers, must be the same and within 5 minutes of the current GMT
time.

**Important** We strongly advise that client library developers make
secure requests using SSL whenever application authentication is used.

### Formats and Protocols

Data returned in the response message is provided in JSON
format. Please refer to the examples provided with each API section
for additional information.

### Error Handling

Although several of the API resources have specific error responses
that correspond to specific situations, there are several generic
error responses that you should be aware of.

Errors are returned as JSON objects that contain "status" and "reason"
attributes. The value of the "status" attribute will always be
"nok". The reason will be an english string that may be, but is not
limited to, one of the following strings.

* Invalid Application
  HTTP Response Code: 500

  A request was made including application identification information,
  but either the application key is invalid or missing.

* Invalid application permissions.
  HTTP Response Code: 500

  A request was made to an API resource that requires a higher
  application permission level.

* Access denied, please contact api-support@blizzard.com
  HTTP Response Code: 500

  The application or IP address has been blocked from making further
  requests. This ban may not be permanent.

* When in doubt, blow it up. (page not found)
  HTTP Response Code: 404

  A request was made to a resource that doesn't exist.

* If at first you don't succeed, blow it up again. (too many requests)
  HTTP Response Code: 500

  The application or IP has been throttled.

* Have you not been through enough? Will you continue to fight what you cannot defeat? (something unexpected happened)
  HTTP Response Code: 500

  There was a server error or equally catastrophic exception
  preventing the request from being fulfilled.

* Invalid authentication header.
  HTTP Response Code: 500

  The application authorization information was mallformed or missing
  when expected.

* Invalid application signature.
  HTTP Response Code: 500

  The application request signature was missing or invalid. This will
  also be thrown if the request date outside of a 15 second window
  from the current GMT time.

*An example API request and and error response*
```plain
GET /api/wow/data/boss/45 HTTP/1.1
Host: us.battle.net
<http headers>
```
```plain
HTTP/1.1 404 Not Found
<http headers>

{"status":"nok", "reason": "When in doubt, blow it up. (page not found)"}
```

### Support

For questions about the API, please use the Community Platform API
forums as a platform to ask questions and get help.

http://us.battle.net/wow/en/forum/2626217/

You can also email
[api-support@blizzard.com](mailto:api-support@blizzard.com) for
matters that you may not want public discussion for.

## API Reference

### Achievement Resources

Achievement APIs provide some simple data about achievements.

#### Achievement

```plain
URL = Host + "/api/wow/achievement/" + AchievementID
```

There are no required query string parameters when accessing this resource.

*An example achievement call*
```plain
GET /api/wow/achievement/2144 HTTP/1.1
Host: us.battle.net
<http headers>
```
```json
{
  "id":2144,
  "title":"What A Long, Strange Trip It's Been",
  "points":50,
  "description":"Complete the world events achievements listed below.",
  "reward":"Rewards: Violet Proto-Drake and Master Riding",
  "rewardItems":[
    {
      "id":44177,
      "name":"Reins of the Violet Proto-Drake",
      "icon":"ability_mount_drake_proto",
      "quality":4,
      "tooltipParams":{}
    }
  ],
  "icon":"achievement_bg_masterofallbgs",
  "criteria":[
    {
      "id":7553,
      "description":"To Honor One's Elders"
    },
    {
      "id":7554,
      "description":"Fool For Love"
    },
    ...
  ],
  accountWide: true
}
```

### Character Resources

Character APIs currently provide character profile information.

#### Profile

The Character Profile API is the primary way to access character
information. This Character Profile API can be used to fetch a single
character at a time through an HTTP GET request to a URL describing
the character profile resource. By default, a basic dataset will be
returned and with each request and zero or more additional fields can
be retrieved. To access this API, craft a resource URL pointing to the
character whos information is to be retrieved.

```plain
URL = Host + "/api/wow/character/" + Realm + "/" + CharacterName

Realm = <proper realm name> | <normalized realm name>
```

There are no required query string parameters when accessing this
resource, although the "fields" query string parameter can optionally
be passed to indicate that one or more of the optional datasets is to
be retrieved. Those additional fields are listed in the subsection
titled "Optional Fields".

*An example Character Profile API request and response.*
```plain
GET /api/wow/character/Medivh/Uther?fields=guild
Host: us.battle.net
HTTP/1.1 200 OK
<http headers>
```

```json
{"realm": "Medivh", "battlegroup": "Ruin", "name": "Uther", "level": 85, "lastModified": 1307596000000,
"thumbnail": "medivh/1/1-avatar.jpg", "race": 1, "achievementPoints": 9745, "gender": 0, "class": 2,
"guild": { ... } }
```

##### Optional Fields

This section contains a list of the optional fields that can be
requested through the mentioned "fields" query string parameter.

* *guild* A summary of the guild that the character belongs to. If the
  character does not belong to a guild and this field is requested,
  this field will not be exposed.

* *stats* A map of character attributes and stats.

* *feed* The activity feed of the character.

* *talents* A list of talent structures.

* *items* A list of items equipted by the character. Use of this field
   will also include the average item level and average item level
   equipped for the character.

* *reputation* A list of the factions that the character has an
   associated reputation with.

* *titles* A list of the titles obtained by the character including
   the currently selected title.

* *professions* A list of the character's professions. It is important
   to note that when this information is retrieved, it will also
   include the known recipes of each of the listed professions.

* *appearance* A map of values that describes the face, features and
   helm/cloak display preferences and attributes.

* *companions* A list of all of the non-combat pets obtained by the
   character.

* *mounts* A list of all of the mounts obtained by the character.

* *pets* A list of all of the combat pets obtained by the character.

* *achievements* A map of achievement data including completion
   timestamps and criteria information.

* *progression* A list of raids and bosses indicating raid progression
   and completedness.

* *pvp* A map of pvp information including arena team membership and
   rated battlegrounds information.

* *quests* A list of quests completed by the character.

*An example Character Profile request with several addtional fields.*
```plain
GET /api/wow/character/Medivh/Uther?fields=guild,items,professions,reputation,stats
Host: us.battle.net
```

###### guild

When a guild is requested, a map is returned with key/value pairs that
describe a basic set of guild information. Note that the rank of the
character is not included in this block as it describes a guild and
not a membership of the guild. To retreive the character's rank within
the guild, you must specific a seperate request to the guild profile
resource.

*An example guild field*
```json
{
  "guild":{
    "name":"Knights of the Silver Hand",
    "realm":"Medivh",
    "level":25,
    "members":50,
    "achievementPoints":7500,
    "emblem":{
      "icon":119,
      "iconColor":"ffb1b8b1",
      "border":0,
      "borderColor":"ffffffff",
      "backgroundColor":"ff006391"
    }
  }
}
```

###### items

When the items field is used, a map structure is returned that
contains information on the equipped items of that character as well
as the average item level of the character.

*An example items field*

```json
{ "items" : { "averageItemLevel" : 353,
      "averageItemLevelEquipped" : 335,
      "back" : { "icon" : "inv_misc_cape_naxxramas_01",
          "id" : 56450,
          "name" : "Azureborne Cloak",
          "quality" : 3,
          "tooltipParams" : {  }
        },
      "chest" : { "icon" : "inv_chest_robe_pvppriest_c_01",
          "id" : 60476,
          "name" : "Vicious Gladiator's Satin Robe",
          "quality" : 4,
          "tooltipParams" : { "enchant" : 4077,
              "gem0" : 52207,
              "gem1" : 52226,
              "set" : [ 60474,
                  60476,
                  60477,
                  60475,
                  60473
                ]
            }
        },
      "feet" : { "icon" : "inv_boots_robe_dungeonrobe_c_03",
          "id" : 63440,
          "name" : "Boots of Lingering Sorrow",
          "quality" : 3,
          "tooltipParams" : { "gem0" : 52117 }
        },
      "finger1" : { "icon" : "inv_jewelry_ring_68",
          "id" : 56333,
          "name" : "Rose Quartz Band",
          "quality" : 3,
          "tooltipParams" : {  }
        },
      "finger2" : { "icon" : "inv_jewelry_ring_75",
          "id" : 56418,
          "name" : "Band of Life Energy",
          "quality" : 3,
          "tooltipParams" : {  }
        },
      "hands" : { "icon" : "inv_gauntlets_robe_pvppriest_c_01",
          "id" : 60473,
          "name" : "Vicious Gladiator's Satin Gloves",
          "quality" : 4,
          "tooltipParams" : { "enchant" : 4068,
              "gem0" : 52245,
              "set" : [ 60474,
                  60476,
                  60477,
                  60475,
                  60473
                ]
            }
        },
      "head" : { "icon" : "inv_helm_robe_pvppriest_c_01",
          "id" : 60474,
          "name" : "Vicious Gladiator's Satin Hood",
          "quality" : 4,
          "tooltipParams" : { "enchant" : 4207,
              "gem0" : 52296,
              "gem1" : 52207,
              "set" : [ 60474,
                  60476,
                  60477,
                  60475,
                  60473
                ]
            }
        },
      "legs" : { "icon" : "inv_pants_robe_pvppriest_c_01",
          "id" : 60475,
          "name" : "Vicious Gladiator's Satin Leggings",
          "quality" : 4,
          "tooltipParams" : { "enchant" : 4112,
              "gem0" : 52207,
              "gem1" : 52245,
              "set" : [ 60474,
                  60476,
                  60477,
                  60475,
                  60473
                ]
            }
        },
      "mainHand" : { "icon" : "inv_staff_13",
          "id" : 65167,
          "name" : "Emberstone Staff",
          "quality" : 3,
          "tooltipParams" : {  }
        },
      "neck" : { "icon" : "inv_jewelry_necklace_44",
          "id" : 70075,
          "name" : "Bloodthirsty Amberjewel Pendant",
          "quality" : 3,
          "tooltipParams" : {  }
        },
      "ranged" : { "icon" : "inv_wand_1h_cataclysm_b_01",
          "id" : 63735,
          "name" : "Darklight Torch",
          "quality" : 2,
          "tooltipParams" : {  }
        },
      "shirt" : { "icon" : "inv_shirt_purple_01",
          "id" : 45037,
          "name" : "Epic Purple Shirt",
          "quality" : 4,
          "tooltipParams" : {  }
        },
      "shoulder" : { "icon" : "inv_shoulder_robe_pvppriest_c_01",
          "id" : 60477,
          "name" : "Vicious Gladiator's Satin Mantle",
          "quality" : 4,
          "tooltipParams" : { "gem0" : 52226,
              "set" : [ 60474,
                  60476,
                  60477,
                  60475,
                  60473
                ]
            }
        },
      "tabard" : { "icon" : "inv_epicguildtabard",
          "id" : 69210,
          "name" : "Renowned Guild Tabard",
          "quality" : 4,
          "tooltipParams" : {  }
        },
      "trinket1" : { "icon" : "spell_shadow_lifedrain",
          "id" : 56351,
          "name" : "Tear of Blood",
          "quality" : 3,
          "tooltipParams" : {  }
        },
      "trinket2" : { "icon" : "inv_jewelry_trinketpvp_01",
          "id" : 18862,
          "name" : "Insignia of the Alliance",
          "quality" : 3,
          "tooltipParams" : {  }
        },
      "waist" : { "icon" : "inv_belt_cloth_raidpriest_i_01",
          "id" : 62386,
          "name" : "Cord of the Raven Queen",
          "quality" : 4,
          "tooltipParams" : { "gem0" : 52244 }
        },
      "wrist" : { "icon" : "inv_bracer_cloth_pvpwarlock_c_01",
          "id" : 60634,
          "name" : "Vicious Gladiator's Cuffs of Prowess",
          "quality" : 4,
          "tooltipParams" : {  }
        }
    } }
```

###### stats

*An example stats field*
```json
{
  "stats":{
    "health":155263,
    "powerType":"mana",
    "power":24732,
    "str":3395,
    "agi":142,
    "sta":8017,
    "int":106,
    "spr":117,
    "attackPower":7025,
    "rangedAttackPower":0,
    "mastery":23.26104,
    "masteryRating":2736,
    "crit":1.351225,
    "critRating":0,
    "hitRating":58,
    "hasteRating":0,
    "expertiseRating":238,
    "spellPower":2133,
    "spellPen":0,
    "spellCrit":3.498852,
    "spellCritRating":0,
    "mana5":1191.0,
    "mana5Combat":1170.0,
    "armor":36505,
    "dodge":12.206712,
    "dodgeRating":1565,
    "parry":13.33957,
    "parryRating":1614,
    "block":57.335938,
    "blockRating":0,
	"pvpResilience": 40,
	"pvpResilienceRating": 0,
    "mainHandDmgMin":2146.0,
    "mainHandDmgMax":2868.0,
    "mainHandSpeed":2.6,
    "mainHandDps":964.1704,
    "mainHandExpertise":10,
    "offHandDmgMin":0.0,
    "offHandDmgMax":0.0,
    "offHandSpeed":2.0,
    "offHandDps":0.0,
    "offHandExpertise":7,
    "rangedDmgMin":-1.0,
    "rangedDmgMax":-1.0,
    "rangedSpeed":-1.0,
    "rangedDps":-1.0,
    "rangedCrit":1.351225,
    "rangedCritRating":0,
    "rangedHitRating":58,
    "pvpPower": 0,
	"pvpPowerRating": 0
  }
}
```

###### feed

*An example feed field*
```json
{
  "feed":[
    {
      "type":"LOOT",
      "timestamp":1335410775000,
      "itemId":77022
    },
    {
      "type":"ACHIEVEMENT",
      "timestamp":1335409252000,
      "achievement":
        {
	  "id":5311,
	  "title":"Elementary",
	  "points":10,
	  "description":"Defeat the Elementium Monstrosity in the Bastion of Twilight while only allowing it to create a single Liquid Ice patch.",
	  "rewardItems":[],
	  "icon":"achievement_dungeon_bastionoftwilight_twilightascendantcouncil",
	  "criteria":[
	    {
	      "id":15471,
	      "description":"Elementium Monstrosity"
	    }
	  ]
	},
      "featOfStrength":false
    }
    {
      "type":"CRITERIA",
      "timestamp":1334115489000,
      "achievement":
        {
	  "id":3016,
	  "title":"In His House He Waits Dreaming (25 player)",
	  "points":10,
	  "description":"Experience all 3 visions of Yogg-Saron's mind in 25-player mode.",
	  "rewardItems":[],
	  "icon":"spell_shadow_brainwash",
	  "criteria":[
	    {
	      "id":10321,
	      "description":"The Assassination of King Llane"
	    },
	    {
	      "id":10322,
	      "description":"The Forging of the Demon Soul"
	    },
	    {
	      "id":10323,
	      "description":"The Tortured Champion"
	    }
	  ]
	},
      "featOfStrength":false,
      "criteria":
        {
	  "id":10322,
	  "description":"The Forging of the Demon Soul"
	}
    },
    {
      "type":"BOSSKILL",
      "timestamp":1334115040000,
      "achievement":
        {
	  "id":2880,
	  "title":"General Vezax kills (Ulduar 25 player)",
	  "points":10,"description":
	  "General Vezax kills (Ulduar 25 player)",
	  "rewardItems":[],
	  "icon":"inv_misc_head_dragon_blue",
	  "criteria":[
	    {
	      "id":9964,"description":"General Vezax"
	    }     
	  ]
	},
      "featOfStrength":false,
      "criteria":
        {
	  "id":9964,
	  "description":"General Vezax"
	},
      "quantity":9,
      "name":"General Vezax"
    }
  }
}
```

###### talents

*An example talents field*
```json
talents: {
  selected: true,
  talents: [
    {
      tier: 0,
      column: 0,
      spell: {
        id: 115173,
        name: "Celerity",
        subtext: "Passive Talent",
        icon: "ability_monk_quipunch",
        description: "Allows you to Roll and Chi Torpedo more often, increases their maximum number of charges by 1, and reduces their cooldown by 5 sec."
      }
    },
    ...
    }
  ],
  glyphs: {
    major: [
      {
        glyph: 1015,
        item: 85685,
        name: "Glyph of Breath of Fire",
        icon: "ability_monk_breathoffire"
      },
      ...
    ],
    minor: [
      {
        glyph: 1041,
        item: 87888,
        name: "Glyph of Fighting Pose",
        icon: "ability_monk_dpsstance"
      },
      ...
    ]
  },
  spec: {
    name: "Windwalker",
    role: "DPS",
    backgroundImage: "bg-monk-battledancer",
    icon: "spell_monk_windwalker_spec",
    description: "A martial artist without peer who pummels foes with hands and fists.",
    order: 2
  },
  calcTalent: "01.01.",
  calcSpec: "b",
  calcGlyph: "Vfp"
},{
  talents: [ ],
  glyphs: {
    major: [ ],
    minor: [ ]
  },
  calcTalent: "",
  calcSpec: "",
  calcGlyph: ""
}
```

###### reputation

*An example reputation field*
```json
{
  "reputation":[
    {
      "id":369,
      "name":"Gadgetzan",
      "standing":5,
      "value":10740,
      "max":12000
    },
    {
      "id":576,
      "name":"Timbermaw Hold",
      "standing":7,
      "value":572,
      "max":999
    },
    {
      "id":470,
      "name":"Ratchet",
      "standing":5,
      "value":10501,
      "max":12000
    },
    {
      "id":59,
      "name":"Thorium Brotherhood",
      "standing":7,
      "value":240,
      "max":999
    },
    {
      "id":1050,
      "name":"Valiance Expedition",
      "standing":7,
      "value":999,
      "max":999
    }
  ]
}
```

###### titles

*An example titles field*
```json
{
  "titles":[
    {
      "id":62,
      "name":"Merciless Gladiator %s"
    },
    {
      "id":79,
      "name":"%s the Diplomat"
    },
    {
      "id":72,
      "name":"Battlemaster %s"
    },
    {
      "id":48,
      "name":"Justicar %s",
      "selected":true
    },
    {
      "id":80,
      "name":"Brutal Gladiator %s"
    },
    {
      "id":53,
      "name":"%s, Champion of the Naaru"
    },
    {
      "id":78,
      "name":"%s the Explorer"
    },
    {
      "id":42,
      "name":"Gladiator %s"
    },
    {
      "id":83,
      "name":"Salty %s"
    },
    {
      "id":90,
      "name":"%s the Malefic"
    }
  ]
}
```

###### professions

*An example professions field*
```json
{
  "professions":{
    "primary":[
      {
        "id":755,
        "name":"Jewelcrafting",
        "icon":"inv_misc_gem_01",
        "rank":525,
        "max":525,
        "recipes":[
          25255,
          25278,
          25280,
          25283,
          25284,
          25287
        ]
      },
      {
        "id":164,
        "name":"Blacksmithing",
        "icon":"trade_blacksmithing",
        "rank":525,
        "max":525,
        "recipes":[
          2660,
          2661,
          2662,
          2663,
          2664
        ]
      }
    ],
    "secondary":[
      {
        "id":129,
        "name":"First Aid",
        "icon":"spell_holy_sealofsacrifice",
        "rank":525,
        "max":525,
        "recipes":[
          3275,
          3276,
          3277,
          3278,
          7928
        ]
      },
      {
        "id":794,
        "name":"Archaeology",
        "icon":"trade_archaeology",
        "rank":406,
        "max":450,
        "recipes":[ ]
      },
      {
        "id":356,
        "name":"Fishing",
        "icon":"trade_fishing",
        "rank":492,
        "max":525,
        "recipes":[ ]
      },
      {
        "id":185,
        "name":"Cooking",
        "icon":"inv_misc_food_15",
        "rank":525,
        "max":525,
        "recipes":[
          2538,
          2539,
          2540,
          2541,
          2542
        ]
      }
    ]
  }
}
```

###### appearance

*An example appearance field*
```json
{
  "appearance":{
    "faceVariation":2,
    "skinColor":0,
    "hairVariation":1,
    "hairColor":8,
    "featureVariation":2,
    "showHelm":true,
    "showCloak":true
  }
}
```

###### companions

*An example companions field*
```json
{
  "companions":[
    4055,
    10673,
    10674,
    10676,
    10677
  ]
}
```

###### mounts

*An example mounts field*
```json
{
  "mounts":[
    30174
  ]
}
```

###### hunterPets

*An example hunterPets fields*
```json
pets: [
  {
    name: "MyPet",
    creature: 57239,
    selected: true,
    slot: 0,
    spec: {
      name: "Tenacity",
      role: "TANK",
      backgroundImage: "bg-deathknight-blood",
      icon: "ability_druid_demoralizingroar",
      description: "",
      order: 1
    },
    calcSpec: "Z"
  },
  ...
]
```

###### achievements

*An example achievements field*
```json
{
  "achievements":{
    "achievementsCompleted":[6,7,8,9,10],
    "achievementsCompletedTimestamp":[1224283700000,1224283700000,1224283700000,1224283700000,1224283700000],
    "criteria":[34,35,36,37,38],
    "criteriaQuantity":[85,85,85,85,85],
    "criteriaTimestamp":[1309580447000,1309580447000,1309580447000,1309580447000,1309580447000],
    "criteriaCreated":[1309580447000,1309580447000,1309580447000,1309580447000,1309580447000]
  }
}
```

###### progression

*An example progression field*
```json
{
  "progression":{
    "raids":[
      {
        "name":"Molten Core",
        "normal":2,
        "heroic":0,
        "id":2717,
        "bosses":[
          {
            "name":"Ragnaros",
            "normalKills":1,
            "heroicKills":0,
            "id":11502
          }
        ]
      },
      {
        "name":"Blackwing Lair",
        "normal":0,
        "heroic":0,
        "id":2677,
        "bosses":[
          {
            "name":"Nefarian",
            "normalKills":0,
            "heroicKills":0,
            "id":11583
          }
        ]
      },
      {
        "name":"Ruins of Ahn'Qiraj",
        "normal":2,
        "heroic":0,
        "id":3429,
        "bosses":[
          {
            "name":"Ossirian the Unscarred",
            "normalKills":-1,
            "heroicKills":0,
            "id":15339
          }
        ]
      },
      {
        "name":"Ahn'Qiraj Temple",
        "normal":2,
        "heroic":0,
        "id":3428,
        "bosses":[
          {
            "name":"C'Thun",
            "normalKills":1,
            "heroicKills":0,
            "id":15727
          }
        ]
      },
      {
        "name":"Karazhan",
        "normal":2,
        "heroic":0,
        "id":3457,
        "bosses":[
          {
            "name":"Prince Malchezaar",
            "normalKills":-1,
            "heroicKills":0,
            "id":15690
          }
        ]
      },
      {
        "name":"Magtheridon's Lair",
        "normal":2,
        "heroic":0,
        "id":3836,
        "bosses":[
          {
            "name":"Magtheridon",
            "normalKills":-1,
            "heroicKills":0,
            "id":17257
          }
        ]
      },
      {
        "name":"Gruul's Lair",
        "normal":2,
        "heroic":0,
        "id":3923,
        "bosses":[
          {
            "name":"Gruul the Dragonkiller",
            "normalKills":-1,
            "heroicKills":0,
            "id":19044
          }
        ]
      },
      {
        "name":"Serpentshrine Cavern",
        "normal":2,
        "heroic":0,
        "id":3607,
        "bosses":[
          {
            "name":"Lady Vashj",
            "normalKills":-1,
            "heroicKills":0,
            "id":21212
          }
        ]
      },
      {
        "name":"Tempest Keep",
        "normal":2,
        "heroic":0,
        "id":3845,
        "bosses":[
          {
            "name":"Kael'thas Sunstrider",
            "normalKills":2,
            "heroicKills":0,
            "id":19622
          }
        ]
      },
      {
        "name":"The Battle for Mount Hyjal",
        "normal":0,
        "heroic":0,
        "id":3606,
        "bosses":[
          {
            "name":"Archimonde",
            "normalKills":0,
            "heroicKills":0,
            "id":17968
          }
        ]
      },
      {
        "name":"Black Temple",
        "normal":2,
        "heroic":0,
        "id":3959,
        "bosses":[
          {
            "name":"Illidan Stormrage",
            "normalKills":1,
            "heroicKills":0,
            "id":22917
          }
        ]
      },
      {
        "name":"The Sunwell",
        "normal":0,
        "heroic":0,
        "id":4075,
        "bosses":[
          {
            "name":"Kil'jaeden",
            "normalKills":0,
            "heroicKills":0,
            "id":25315
          }
        ]
      },
      {
        "name":"Vault of Archavon",
        "normal":2,
        "heroic":0,
        "id":4603,
        "bosses":[
          {
            "name":"Archavon the Stone Watcher",
            "normalKills":8,
            "heroicKills":0,
            "id":31125
          },
          {
            "name":"Emalon the Storm Watcher",
            "normalKills":2,
            "heroicKills":0,
            "id":33993
          },
          {
            "name":"Koralon the Flame Watcher",
            "normalKills":5,
            "heroicKills":0,
            "id":35013
          },
          {
            "name":"Toravon the Ice Watcher",
            "normalKills":5,
            "heroicKills":0,
            "id":38433
          }
        ]
      },
      {
        "name":"Naxxramas",
        "normal":2,
        "heroic":0,
        "id":3456,
        "bosses":[
          {
            "name":"Anub'Rekhan",
            "normalKills":9,
            "heroicKills":0,
            "id":15956
          },
          {
            "name":"Grand Widow Faerlina",
            "normalKills":6,
            "heroicKills":0,
            "id":15953
          },
          {
            "name":"Maexxna",
            "normalKills":7,
            "heroicKills":0,
            "id":15952
          },
          {
            "name":"Patchwerk",
            "normalKills":9,
            "heroicKills":0,
            "id":16028
          },
          {
            "name":"Grobbulus",
            "normalKills":8,
            "heroicKills":0,
            "id":15931
          },
          {
            "name":"Gluth",
            "normalKills":7,
            "heroicKills":0,
            "id":15932
          },
          {
            "name":"Thaddius",
            "normalKills":7,
            "heroicKills":0,
            "id":15928
          },
          {
            "name":"Noth the Plaguebringer",
            "normalKills":10,
            "heroicKills":0,
            "id":15954
          },
          {
            "name":"Heigan the Unclean",
            "normalKills":9,
            "heroicKills":0,
            "id":15936
          },
          {
            "name":"Loatheb",
            "normalKills":9,
            "heroicKills":0,
            "id":16011
          },
          {
            "name":"Instructor Razuvious",
            "normalKills":8,
            "heroicKills":0,
            "id":16061
          },
          {
            "name":"Gothik the Harvester",
            "normalKills":7,
            "heroicKills":0,
            "id":16060
          },
          {
            "name":"The Four Horsemen",
            "normalKills":7,
            "heroicKills":0,
            "id":59450
          },
          {
            "name":"Sapphiron",
            "normalKills":8,
            "heroicKills":0,
            "id":15989
          },
          {
            "name":"Kel'Thuzad",
            "normalKills":8,
            "heroicKills":0,
            "id":15990
          }
        ]
      },
      {
        "name":"The Obsidian Sanctum",
        "normal":2,
        "heroic":0,
        "id":4493,
        "bosses":[
          {
            "name":"Sartharion",
            "normalKills":9,
            "heroicKills":0,
            "id":28860
          }
        ]
      },
      {
        "name":"The Eye of Eternity",
        "normal":2,
        "heroic":0,
        "id":4500,
        "bosses":[
          {
            "name":"Malygos",
            "normalKills":9,
            "heroicKills":0,
            "id":28859
          }
        ]
      },
      {
        "name":"Ulduar",
        "normal":1,
        "heroic":0,
        "id":4273,
        "bosses":[
          {
            "name":"Flame Leviathan",
            "normalKills":6,
            "heroicKills":0,
            "id":33113
          },
          {
            "name":"Razorscale",
            "normalKills":5,
            "heroicKills":0,
            "id":33186
          },
          {
            "name":"XT-002 Deconstructor",
            "normalKills":2,
            "heroicKills":0,
            "id":33293
          },
          {
            "name":"Ignis the Furnace Master",
            "normalKills":2,
            "heroicKills":0,
            "id":33118
          },
          {
            "name":"Assembly of Iron",
            "normalKills":0,
            "heroicKills":0,
            "id":65195
          },
          {
            "name":"Kologarn",
            "normalKills":0,
            "heroicKills":0,
            "id":32930
          },
          {
            "name":"Auriaya",
            "normalKills":1,
            "heroicKills":0,
            "id":33515
          },
          {
            "name":"Hodir",
            "normalKills":0,
            "heroicKills":0,
            "id":64899
          },
          {
            "name":"Thorim",
            "normalKills":0,
            "heroicKills":0,
            "id":64985
          },
          {
            "name":"Freya",
            "normalKills":0,
            "heroicKills":0,
            "id":65074
          },
          {
            "name":"Leviathan Mk II",
            "normalKills":0,
            "heroicKills":0,
            "id":33432
          },
          {
            "name":"General Vezax",
            "normalKills":0,
            "heroicKills":0,
            "id":33271
          },
          {
            "name":"Yogg-Saron",
            "normalKills":0,
            "heroicKills":0,
            "id":33288
          },
          {
            "name":"Algalon the Observer",
            "normalKills":0,
            "heroicKills":0,
            "id":65184
          }
        ]
      },
      {
        "name":"Onyxia's Lair",
        "normal":2,
        "heroic":0,
        "id":2159,
        "bosses":[
          {
            "name":"Onyxia",
            "normalKills":4,
            "heroicKills":0,
            "id":10184
          }
        ]
      },
      {
        "name":"Trial of the Crusader",
        "normal":2,
        "heroic":1,
        "id":4722,
        "bosses":[
          {
            "name":"Icehowl",
            "normalKills":6,
            "heroicKills":3,
            "id":34797
          },
          {
            "name":"Lord Jaraxxus",
            "normalKills":6,
            "heroicKills":3,
            "id":34780
          },
          {
            "name":"Defeat the Faction Champions",
            "normalKills":5,
            "heroicKills":1,
            "id":68184
          },
          {
            "name":"Eydis Darkbane",
            "normalKills":5,
            "heroicKills":1,
            "id":34496
          },
          {
            "name":"Anub'arak",
            "normalKills":5,
            "heroicKills":0,
            "id":34564
          }
        ]
      },
      {
        "name":"Icecrown Citadel",
        "normal":2,
        "heroic":0,
        "id":4812,
        "bosses":[
          {
            "name":"Lord Marrowgar",
            "normalKills":18,
            "heroicKills":0,
            "id":36612
          },
          {
            "name":"Lady Deathwhisper",
            "normalKills":18,
            "heroicKills":0,
            "id":36855
          },
          {
            "name":"Claim victory in the Gunship Battle",
            "normalKills":18,
            "heroicKills":0,
            "id":72959
          },
          {
            "name":"The Deathbringer",
            "normalKills":16,
            "heroicKills":0,
            "id":72928
          },
          {
            "name":"Festergut",
            "normalKills":13,
            "heroicKills":0,
            "id":36626
          },
          {
            "name":"Rotface",
            "normalKills":11,
            "heroicKills":0,
            "id":36627
          },
          {
            "name":"Professor Putricide",
            "normalKills":6,
            "heroicKills":0,
            "id":36678
          },
          {
            "name":"Prince Valanar",
            "normalKills":9,
            "heroicKills":0,
            "id":37970
          },
          {
            "name":"Blood-Queen Lana'thel",
            "normalKills":5,
            "heroicKills":0,
            "id":37955
          },
          {
            "name":"Rescue Valithria Dreamwalker",
            "normalKills":4,
            "heroicKills":0,
            "id":72706
          },
          {
            "name":"Sindragosa",
            "normalKills":2,
            "heroicKills":0,
            "id":36853
          },
          {
            "name":"The Lich King",
            "normalKills":1,
            "heroicKills":0,
            "id":36597
          }
        ]
      },
      {
        "name":"The Ruby Sanctum",
        "normal":0,
        "heroic":0,
        "id":4987,
        "bosses":[
          {
            "name":"Halion",
            "normalKills":0,
            "heroicKills":0,
            "id":39863
          }
        ]
      },
      {
        "name":"Baradin Hold",
        "normal":0,
        "heroic":0,
        "id":5600,
        "bosses":[
          {
            "name":"Argaloth",
            "normalKills":0,
            "heroicKills":0,
            "id":47120
          },
          {
            "name":"Occu'thar",
            "normalKills":0,
            "heroicKills":0,
            "id":52363
          }
        ]
      },
      {
        "name":"Blackwing Descent",
        "normal":0,
        "heroic":0,
        "id":5094,
        "bosses":[
          {
            "name":"Magmaw",
            "normalKills":0,
            "heroicKills":0,
            "id":41570
          },
          {
            "name":"Toxitron",
            "normalKills":0,
            "heroicKills":0,
            "id":42180
          },
          {
            "name":"Maloriak",
            "normalKills":0,
            "heroicKills":0,
            "id":41378
          },
          {
            "name":"Atramedes",
            "normalKills":0,
            "heroicKills":0,
            "id":41442
          },
          {
            "name":"Chimaeron",
            "normalKills":0,
            "heroicKills":0,
            "id":43296
          },
          {
            "name":"Nefarian",
            "normalKills":0,
            "heroicKills":0,
            "id":41376
          }
        ]
      },
      {
        "name":"The Bastion of Twilight",
        "normal":0,
        "heroic":0,
        "id":5334,
        "bosses":[
          {
            "name":"Halfus Wyrmbreaker",
            "normalKills":0,
            "heroicKills":0,
            "id":44600
          },
          {
            "name":"Valiona",
            "normalKills":0,
            "heroicKills":0,
            "id":45992
          },
          {
            "name":"Elementium Monstrosity",
            "normalKills":0,
            "heroicKills":0,
            "id":43735
          },
          {
            "name":"Cho'gall",
            "normalKills":0,
            "heroicKills":0,
            "id":43324
          },
          {
            "name":"Sinestra",
            "normalKills":0,
            "heroicKills":0,
            "id":45213
          }
        ]
      },
      {
        "name":"Throne of the Four Winds",
        "normal":0,
        "heroic":0,
        "id":5638,
        "bosses":[
          {
            "name":"Conclave of Wind",
            "normalKills":0,
            "heroicKills":0,
            "id":88835
          },
          {
            "name":"Al'Akir",
            "normalKills":0,
            "heroicKills":0,
            "id":46753
          }
        ]
      },
      {
        "name":"Firelands",
        "normal":0,
        "heroic":0,
        "id":5723,
        "bosses":[
          {
            "name":"Beth'tilac",
            "normalKills":0,
            "heroicKills":0,
            "id":52498
          },
          {
            "name":"Lord Rhyolith",
            "normalKills":0,
            "heroicKills":0,
            "id":53772
          },
          {
            "name":"Alysrazor",
            "normalKills":0,
            "heroicKills":0,
            "id":52530
          },
          {
            "name":"Shannox",
            "normalKills":0,
            "heroicKills":0,
            "id":53691
          },
          {
            "name":"Baleroc",
            "normalKills":0,
            "heroicKills":0,
            "id":53494
          },
          {
            "name":"Majordomo Staghelm",
            "normalKills":0,
            "heroicKills":0,
            "id":52571
          },
          {
            "name":"Ragnaros",
            "normalKills":0,
            "heroicKills":0,
            "id":102237
          },
          {
            "name":"Ragnaros",
            "normalKills":0,
            "heroicKills":0,
            "id":52409
          }
        ]
      }
    ]
  }
}
```

###### pvp

*An example pvp field*
```json
"pvp":{
    "ratedBattlegrounds":{
      "personalRating": 3100,
      "battlegrounds":[
        {
          "name":"Arathi Basin",
          "played":98745,
          "won":98744
        },
        {
          "name":"The Battle for Gilneas",
          "played":0,
          "won":0
        },
        {
          "name":"Eye of the Storm",
          "played":0,
          "won":0
        },
        {
          "name":"Strand of the Ancients",
          "played":0,
          "won":0
        },
        {
          "name":"Twin Peaks",
          "played":0,
          "won":0
        },
        {
          "name":"Warsong Gulch",
          "played":0,
          "won":0
        }
      ]
    },
    "arenaTeams":[
      {
        "name":"Lordaeron's Defense",
        "personalRating":2700,
        "teamRating":2700,
        "size":"5v5"
      }
    ],
    "totalHonorableKills":98475
  }
```

###### quests

*An example quests field*
```json
{
  "quests":[
    26977,
    26997,
    27044,
    27060,
    27064,
    27072,
    27092,
    27106,
    28238,
    28596,
    28807,
    28832
  ]
}
```

### Guild Resources

Guild APIs currently provide guild profile information.

#### Profile API

The guild profile API is the primary way to access guild
information. This guild profile API can be used to fetch a single
guild at a time through an HTTP GET request to a url describing the
guild profile resource. By default, a basic dataset will be returned
and with each request and zero or more additional fields can be
retrieved. To access this API, craft a resource URL pointing to the
guild whos information is to be retrieved.

```plain
URL = Host + "/api/wow/guild/" + Realm + "/" + GuildName

Realm = <proper realm name> | <normalized realm name>
```

There are no required query string parameters when accessing this
resource, although the "fields" query string parameter can optionally
be passed to indicate that one or more of the optional datasets is to
be retrieved. Those additional fields are listed in the subsection
titled "Optional Fields".

*An example Guild Profile request and response.*
```plain
GET /api/wow/guild/Medivh/Knights%20of%20the%20Silver%20Hand
Host: us.battle.net
HTTP/1.1 200 OK
<http headers>
```
```json
{ "name":"Knights of the Silver Hand", "level":25, "realm":"Bronzebeard", "battlegroup":"Ruin", "side":0, "achievementPoints":800 }
```

The core dataset returned includes the guild's name, level, faction
and achievement points.

##### Optional Fields

This section contains a list of the optional fields that can be
requested.

* *members* A list of characters that are a member of the guild

* *achievements* A set of data structures that describe the
   achievements earned by the guild.

* *news* A set of data structures that describe the news feed of the
   guild.

*An example Guild Profile request with several addtional fields.*
```plain
GET /api/wow/guild/Medivh/Knights%20of%20the%20Silver%20Hand?fields=achievements,members
Host: us.battle.net
```

###### members

When the members list is requested, a list of character objects is
returned. Each object in the returned members list contains a
character block as well as a rank field.

*An example members block*
```json
"members":[
    {
      "character":{
        "name":"Mequieres",
        "realm":"Medivh",
        "class":5,
        "race":1,
        "gender":"female",
        "level":85,
        "achievementPoints":6910,
        "thumbnail":"medivh/66/3930434-avatar.jpg"
      },
      "rank":1
    },
    {
      "character":{
        "name":"Berex",
        "realm":"Medivh",
        "class":2,
        "race":1,
        "gender":"male",
        "level":81,
        "achievementPoints":3410,
        "thumbnail":"medivh/159/3931551-avatar.jpg"
      },
      "rank":2
    },
    {
      "character":{
        "name":"Avalos",
        "realm":"Medivh",
        "class":2,
        "race":1,
        "gender":"male",
        "level":85,
        "achievementPoints":9755,
        "thumbnail":"medivh/2/4022530-avatar.jpg"
      },
      "rank":1
    },
    {
      "character":{
        "name":"Getafixes",
        "realm":"Medivh",
        "class":11,
        "race":4,
        "gender":"male",
        "level":85,
        "achievementPoints":7225,
        "thumbnail":"medivh/220/4064988-avatar.jpg"
      },
      "rank":7
    },
    {
      "character":{
        "name":"Thomaslv",
        "realm":"Medivh",
        "class":2,
        "race":1,
        "gender":"male",
        "level":85,
        "achievementPoints":5130,
        "thumbnail":"medivh/61/4090173-avatar.jpg"
      },
      "rank":6
    },
    {
      "character":{
        "name":"Jeanelly",
        "realm":"Medivh",
        "class":8,
        "race":7,
        "gender":"female",
        "level":85,
        "achievementPoints":9175,
        "thumbnail":"medivh/16/11635728-avatar.jpg"
      },
      "rank":3
    },
    {
      "character":{
        "name":"Shaynk",
        "realm":"Medivh",
        "class":4,
        "race":22,
        "gender":"male",
        "level":85,
        "achievementPoints":7005,
        "thumbnail":"medivh/13/11885837-avatar.jpg"
      },
      "rank":1
    },
    {
      "character":{
        "name":"Odette",
        "realm":"Medivh",
        "class":7,
        "race":11,
        "gender":"female",
        "level":85,
        "achievementPoints":3335,
        "thumbnail":"medivh/214/14322390-avatar.jpg"
      },
      "rank":8
    },
    {
      "character":{
        "name":"Addeon",
        "realm":"Medivh",
        "class":9,
        "race":7,
        "gender":"male",
        "level":85,
        "achievementPoints":4755,
        "thumbnail":"medivh/11/16220683-avatar.jpg"
      },
      "rank":3
    },
    {
      "character":{
        "name":"Tinkerton",
        "realm":"Medivh",
        "class":4,
        "race":7,
        "gender":"male",
        "level":85,
        "achievementPoints":7755,
        "thumbnail":"medivh/7/50893575-avatar.jpg"
      },
      "rank":3
    },
    {
      "character":{
        "name":"Itaachi",
        "realm":"Medivh",
        "class":8,
        "race":7,
        "gender":"female",
        "level":85,
        "achievementPoints":5975,
        "thumbnail":"medivh/154/74538650-avatar.jpg"
      },
      "rank":3
    },
    {
      "character":{
        "name":"Korale",
        "realm":"Medivh",
        "class":5,
        "race":22,
        "gender":"male",
        "level":85,
        "achievementPoints":5620,
        "thumbnail":"medivh/88/81679704-avatar.jpg"
      },
      "rank":7
    }
  ]
```

###### achievements

When requesting achievement data, several sets of data will be
returned.

* *achievementsCompleted* A list of achievement ids.

* *achievementsCompletedTimestamp* A list of timestamps whose places
   correspond to the achievement ids in the "achievementsCompleted"
   list. The value of each timestamp indicates when the related
   achievement was earned by the guild.

* *criteria* A list of criteria ids that can be used to determine the
   partial completedness of guild achievements.

* *criteriaQuantity* A list of values associated with a given
   achievement criteria. The position of a value corresponds to the
   position of a given achivement criteria.

* *criteriaTimestamp* A list of timestamps where the value represents
   when the criteria was considered complete. The position of a value
   corresponds to the position of a given achivement criteria.

* *criteriaCreated* A list of timestamps where the value represents
   when the criteria was considered started. The position of a value
   corresponds to the position of a given achivement criteria.

*An example achievements block*
```json
"achievements":{
    "achievementsCompleted":[4860,4861,4912,4913,4943],
    "achievementsCompletedTimestamp":[1305087854000,1292834684000,1307514307000,1296629625000,1292837842000],
    "criteria":[13756,13757,13835,13864,13865],
    "criteriaQuantity":[25,500300151,1,525,525],
    "criteriaTimestamp":[1307514307000,1296629625000,1292834684000,1292744289000,1293154824000],
    "criteriaCreated":[1291708917000,1291716646000,1292834684000,1292573188000,1292728137000]
  }
```

###### news

When the news feed is requested, you receive a list of news
objects. Each one has a type, a timestamp, and then some other data
depending on the type: itemId, achievement object, etc.

*An example news block*
```json
{
  "news":[
    {
      "type":"guildCreated",
      "timestamp":1335394380000
    },
    {
      "type":"itemLoot",
      "character":"David",
      "timestamp":1335737040000,
      "itemId":72833
    },
    {
      "type":"itemPurchase",
      "character":"Kim",
      "timestamp":1335733380000,
      "itemId":71283
    },
    {
      "type":"guildLevel",
      "timestamp":1335394380000,
      "levelUp":20
    },
    {
      "type":"guildAchievement",
      "character":"Dustin",
      "timestamp":1335394380000,
      "achievement":
        {
	  "id":4945,
	  "title":"Guild Level 15",
	  "points":10,
	  "description":"Reach guild level 15.",
	  "reward":"Reward: Wrap of Unity",
	  "rewardItems":[
	    {
	      "id":63206,
	      "name":"Wrap of Unity",
	      "icon":"inv_guild_cloak_alliance_b",
	      "quality":3,
	      "tooltipParams":{}
	    },
	    {
	      "id":63207,
	      "name":"Wrap of Unity",
	      "icon":"inv_guild_cloak_horde_b",
	      "quality":3,
	      "tooltipParams":{}
	    }
	  ],
	  "icon":"achievement_guild_level15",
	  "criteria":[
	    {
	      "id":13876,
	      "description":"Reach guild level 15."
	    }
	  ]
	}
    },
    {
      "type":"playerAchievement",
      "character":"Mike",
      "timestamp":1335763860000,
      "achievement":
        {
	  "id":2144,
	  "title":"What A Long, Strange Trip It's Been",
	  "points":50,
	  "description":"Complete the world events achievements listed below.",
	  "reward":"Rewards: Violet Proto-Drake and Master Riding",
	  "rewardItems":[
	    {
	      "id":44177,
	      "name":"Reins of the Violet Proto-Drake",
	      "icon":"ability_mount_drake_proto",
	      "quality":4,
	      "tooltipParams":{}
	    }
	  ],
	  "icon":"achievement_bg_masterofallbgs",
	  "criteria":[
	    {
	      "id":7553,
	      "description":"To Honor One's Elders"
	    },
	    {
	      "id":7554,
	      "description":"Fool For Love"
	    },
	    {
	      "id":9879,
	      "description":"Noble Gardener"
	    },
	    {
	      "id":7555,
	      "description":"For The Children"
	    },
	    {
	      "id":7556,
	      "description":"The Flame Warden"
	    },
	    {
	      "id":7557,
	      "description":"Brewmaster"
	    },
	    {
	      "id":7558,
	      "description":"Hallowed Be Thy Name"
	    },
	    {
	      "id":7559,
	      "description":"Merrymaker"
	    }
	  ]
	}
    }
  ]
}
```

### Realm Resources

Realm APIs currently provide realm status information.

#### Realm Status

The realm status API allows developers to retrieve realm status
information. This information is limited to whether or not the realm
is up, the type and state of the realm, the current population, and
the status of the two world pvp zones.

```plain
URL = Host + "/api/wow/realm/status"
```

There are no required query string parameters when accessing this
resource, although the "realms" query string parameter can optionally
be passed to limit the realms returned to one or more.

##### Pvp Area Status Fields

* *area* An internal id of this zone.

* *controlling-faction* Which faction is controlling the zone at the
   moment.

* *status* The current status of the zone. The possible values are

  * -1: Unknown
  * 0: Idle
  * 1: Populating
  * 2: Active
  * 3: Concluded

* *next* A timestamp of when the next battle starts.

*An example Realm Status request and response*
```plain
GET /api/wow/realm/status?realms=Medivh,Lightbringer HTTP/1.1
Host: us.battle.net
```
```json
{
  "realms":[
    {
      "type":"pve",
      "queue":false,
      "wintergrasp":{
        "area":1,
	"controlling-faction":1,
	"status":0,
	"next":1334885402591
      },
      "tol-barad":{
        "area":21,
	"controlling-faction":1,
	"status":0,
	"next":1334884311692
      },
      "status":true,
      "population":"high",
      "name":"Lightbringer",
      "slug":"lightbringer"
    },
    {
      "type":"pve",
      "queue":false,
      "wintergrasp":{
        "area":1,
	"controlling-faction":0,
	"status":0,
	"next":1334885402591
      },
      "tol-barad":{
        "area":21,
	"controlling-faction":0,
	"status":0,
	"next":1334884311692
      },
      "status":true,
      "population":"medium",
      "name":"Medivh",
      "slug":"medivh"
    }
  ]
}
```

### Recipe Resources

Recipe APIs currently provide recipe information.

#### Recipe API

The recipe API provides basic recipe information.

```plain
URL = Host + "/api/wow/recipe/" + RecipeId
```

*An example recipe API request and response*
```plain
GET /api/wow/recipe/33994 HTTP/1.1
Host: us.battle.net
```
```json
{
    "id": 33994,
    "name": "Enchant Gloves - Precise Strikes",
    "profession": "Enchanting",
    "icon": "spell_holy_greaterheal"
}
```

### Auction Resources

Auction APIs currently provide rolling batches of data about current
auctions. Fetching auction dumps is a two step process that involves
checking a per-realm index file to determine if a recent dump has been
generated and then fetching the most recently generated dump file if
necessary.

#### Current Auctions APIs

This API resource provides a per-realm list of recently generated
auction house data dumps.

```plain
URL = Host + "/api/wow/auction/data/" + realm
```

There are no required query string parameters when accessing this
resource.

*An example Current Auctions request and response*
```plain
GET /api/wow/auction/data/medivh HTTP/1.1
Host: us.battle.net
```
```json
{
  "files":[
    {
      "url":"http://us.battle.net/auction-data/medivh/auctions-1311362443895.json.gz",
      "lastModified":1311362443895
    }
  ]
}
```

#### Current Auctions Data

The current auctions data is represented as JSON structures containing
auction data for the three auctions houses available on each realm.

```json
{
    "realm": {
        "name": "Medivh",
        "slug": "medivh"
    },
    "alliance": {
        "auctions": [
            {
                "auc": 500,
                "item": 49284,
                "owner": "Uther",
                "bid": 150000,
                "buyout": 450000,
                "quantity": 11,
                "timeLeft": "VERY_LONG"
            },
            {
                "auc": 504,
                "item": 2160,
                "owner": "Ronakada",
                "bid": 140000,
                "buyout": 150000,
                "quantity": 1,
                "timeLeft": "LONG"
            }
        ]
    },
    "horde": {
        "auctions": [
            {
                "auc": 501,
                "item": 44575,
                "owner": "Thrall",
                "bid": 26751,
                "buyout": 57665,
                "quantity": 1
                "timeLeft": "MEDIUM"
            }
        ]
    },
    "neutral": {
        "auctions": [
            {
                "auc": 502,
                "item": 63271,
                "owner": "Arthas",
                "bid": 20000,
                "buyout": 50000,
                "quantity": 1
                "timeLeft": "SHORT"
            }
        ]
    }
}
```

### Item Resources

Item APIs currently provide item information.

#### Item API

The item API provides detailed item information. This includes item
set information if this item is part of a set.

```plain
URL = Host + "/api/wow/item/" + ItemId
```

*An example Item API request and response*
```plain
GET /api/wow/item/38268 HTTP/1.1
Host: us.battle.net
```
```json
{
  "id":38268,
  "disenchantingSkillRank":-1,
  "description":"Give to a Friend",
  "name":"Spare Hand",
  "stackable":1,
  "itemBind": 0,
  "bonusStats":[],
  "itemSpells":[],
  "buyPrice":12,
  "itemClass": 2,
  "itemSubClass": 14,
  "containerSlots":0,
  "weaponInfo":{
    "damage":[
      {
        "minDamage":1,
        "maxDamage":2
      }
    ],
    "weaponSpeed":2.5,
    "dps":0.6
  },
  "inventoryType":13,
  "equippable":true,
  "itemLevel":1,
  "maxCount":0,
  "maxDurability":16,
  "minFactionId":0,
  "minReputation":0,
  "quality":0,
  "sellPrice":2,
  "requiredLevel":70,
  "requiredSkill":0,
  "requiredSkillRank":0,
  "itemSource":{
    "sourceId":0,
    "sourceType":"NONE"
  },
  "baseArmor":0,
  "hasSockets":false,
  "isAuctionable":true
}
```

#### Item Set API

The item set API provides detailed item set information.

```plain
URL = Host + "/api/wow/item/set/" + SetId
```

*An example Item Set API request and response*
```plain
GET /api/wow/item/set/1060 HTTP/1.1
Host: us.battle.net
```
```json
{
  "id":1060,
  "name":"Deep Earth Vestments",
  "setBonuses":[
    {
      "description":"After using Innervate, the mana cost of your healing spells is reduced by 25% for 15 sec.",
      "threshold":2
    },
    {
      "description":"Your Rejuvenation and Regrowth spells have a 10% chance to Timeslip and have double the normal duration.",
      "threshold":4
    }
  ]
}
```

### PVP Resources

PVP APIs currently provide arena team and ladder information.

#### Arena Team API

The Arena Team API provides detailed arena team information.

```plain
TeamSize = "2v2" | "3v3" | "5v5"
URL = Host + "/api/wow/arena/" + Realm + "/" + TeamSize + "/" + TeamName
```

*An example Arena Team API request and response*
```plsin
GET /api/wow/arena/bonechewer/2v2/Samurai%20Jack HTTP/1.1
Host: us.battle.net
```
```json
{
  "realm": "Bonechewer",
  "ranking":3,
  "rating":2407,
  "teamsize":2,
  "created":"2010-12-14",
  "name":"Samurai Jack",
  "gamesPlayed":0,
  "gamesWon":0,
  "gamesLost":0,
  "sessionGamesPlayed":84,
  "sessionGamesWon":65,
  "sessionGamesLost":19,
  "lastSessionRanking":49,
  "side":"horde",
  "currentWeekRanking":3,
  "members":[
    {
      "character":{
        "name":"Enzi",
        "realm":"Bonechewer",
        "class":5,
        "race":5,
        "gender":0,
        "level":85,
        "achievementPoints":7825,
        "thumbnail":"bonechewer/63/50019391-avatar.jpg"
      },
      "rank":1,
      "gamesPlayed":0,
      "gamesWon":0,
      "gamesLost":0,
      "sessionGamesPlayed":79,
      "sessionGamesWon":61,
      "sessionGamesLost":18,
      "personalRating":2407
    },
    {
      "character":{
        "name":"Treeofapples",
        "realm":"Bonechewer",
        "class":11,
        "race":6,
        "gender":0,
        "level":85,
        "achievementPoints":4755,
        "thumbnail":"bonechewer/31/56730399-avatar.jpg"
      },
      "rank":0,
      "gamesPlayed":0,
      "gamesWon":0,
      "gamesLost":0,
      "sessionGamesPlayed":84,
      "sessionGamesWon":65,
      "sessionGamesLost":19,
      "personalRating":2407
    }
  ]
}
```

#### Arena Ladder API

The Arena Team Ladder API provides arena ladder information for a
battlegroup.

```plain
TeamSize = "2v2" | "3v3" | "5v5"
URL = Host + "/api/wow/pvp/arena/" + Battlegroup + "/" + TeamSize
```

Optional Query String Parameters

* *page* Which page of results to return (defaults to 1)

* *size* How many results to return per page (defaults to 50)

* *asc* Whether to return the results in ascending order. Defaults to
   "true", accepts "true" or "false"

*An example Arena Team Ladder API request and response*
```plain
GET /api/wow/pvp/arena/ruin/2v2 HTTP/1.1
Host: us.battle.net
```
```json
{
  "arenateam":[
    {
      "realm":"Mannoroth",
      "ranking":1,
      "rating":2652,
      "teamsize":2,
      "created":"2011-07-10",
      "name":"Why is animal on my W",
      "gamesPlayed":0,
      "gamesWon":0,
      "gamesLost":0,
      "sessionGamesPlayed":95,
      "sessionGamesWon":82,
      "sessionGamesLost":13,
      "lastSessionRanking":0,
      "side":"alliance",
      "currentWeekRanking":0,
      "members":[
        {
          "character":{
            "name":"Soysauce",
            "realm":"Mannoroth",
            "class":6,
            "race":1,
            "gender":1,
            "level":85,
            "achievementPoints":1850,
            "thumbnail":"mannoroth/63/76583999-avatar.jpg"
          },
          "rank":0,
          "gamesPlayed":0,
          "gamesWon":0,
          "gamesLost":0,
          "sessionGamesPlayed":0,
          "sessionGamesWon":0,
          "sessionGamesLost":0,
          "personalRating":1000
        }
      ]
    },
    {
      "realm":"Mannoroth",
      "ranking":2,
      "rating":2602,
      "teamsize":2,
      "created":"2011-01-02",
      "name":"WHY'S ANIMAL ON MY FACE",
      "gamesPlayed":0,
      "gamesWon":0,
      "gamesLost":0,
      "sessionGamesPlayed":100,
      "sessionGamesWon":88,
      "sessionGamesLost":12,
      "lastSessionRanking":4,
      "side":"alliance",
      "currentWeekRanking":0,
      "members":[
        {
          "character":{
            "name":"Xeti",
            "realm":"Mannoroth",
            "class":7,
            "race":3,
            "gender":1,
            "level":85,
            "achievementPoints":4520,
            "thumbnail":"mannoroth/157/95140253-avatar.jpg"
          },
          "rank":0,
          "gamesPlayed":0,
          "gamesWon":0,
          "gamesLost":0,
          "sessionGamesPlayed":0,
          "sessionGamesWon":0,
          "sessionGamesLost":0,
          "personalRating":1000
        },
        {
          "character":{
            "name":"Nvo",
            "realm":"Mannoroth",
            "class":2,
            "race":3,
            "gender":1,
            "level":85,
            "achievementPoints":4670,
            "thumbnail":"mannoroth/213/93150677-avatar.jpg"
          },
          "rank":1,
          "gamesPlayed":0,
          "gamesWon":0,
          "gamesLost":0,
          "sessionGamesPlayed":0,
          "sessionGamesWon":0,
          "sessionGamesLost":0,
          "personalRating":1000
        }
      ]
    },
    {
      "realm":"Shattered Hand",
      "ranking":3,
      "rating":2471,
      "teamsize":2,
      "created":"2011-07-05",
      "name":"Lord Slug Zero Dmg",
      "gamesPlayed":18,
      "gamesWon":16,
      "gamesLost":2,
      "sessionGamesPlayed":155,
      "sessionGamesWon":130,
      "sessionGamesLost":25,
      "lastSessionRanking":0,
      "side":"horde",
      "currentWeekRanking":1,
      "members":[
        {
          "character":{
            "name":"L�rdsl�g",
            "realm":"Shattered Hand",
            "class":9,
            "race":2,
            "gender":1,
            "level":85,
            "achievementPoints":4925,
            "thumbnail":"shattered-hand/121/93928313-avatar.jpg"
          },
          "rank":0,
          "gamesPlayed":18,
          "gamesWon":16,
          "gamesLost":2,
          "sessionGamesPlayed":152,
          "sessionGamesWon":130,
          "sessionGamesLost":22,
          "personalRating":2473
        },
        {
          "character":{
            "name":"Ohhyea",
            "realm":"Shattered Hand",
            "class":11,
            "race":6,
            "gender":0,
            "level":85,
            "achievementPoints":14395,
            "thumbnail":"shattered-hand/122/50562426-avatar.jpg"
          },
          "rank":1,
          "gamesPlayed":18,
          "gamesWon":16,
          "gamesLost":2,
          "sessionGamesPlayed":154,
          "sessionGamesWon":130,
          "sessionGamesLost":24,
          "personalRating":2471
        }
      ]
    }
  ]
}
```

#### Rated Battleground Ladder API

The Rated Battleground Ladder API provides ladder information for a region.

```plain
URL = Host + "/api/wow/pvp/ratedbg/ladder"
```

Optional Query String Parameters

* *page* Which page of results to return (defaults to 1)

* *size* How many results to return per page (defaults to 50)

* *asc* Whether to return the results in ascending order. Defaults to
  "true", accepts "true" or "false"
  
*An example Rated Battleground Ladder API request and response*
```plain
GET /api/wow/pvp/ratedbg/ladder HTTP/1.1
Host: us.battle.net
```
```json
{
  "bgRecord":[
    {
      "rank":1,
      "bgRating":3021,
      "wins":290,
      "losses":32,
      "played":322,
      "realm":
      {
        "name":"Kel'Thuzad",
  "slug":"kelthuzad",
  "battlegroup":"Nightfall"
      },
      "battlegroup":
      {
        "name":"Nightfall",
  "slug":"nightfall"
      },
      "character":
      {
        "name":"Player1",
  "realm":"Kel'Thuzad",
  "class":8,
  "race":1,
  "gender":1,
  "level":85,
  "achievementPoints":5550,
  "thumbnail":"kelthuzad/173/xxx-avatar.jpg"
      },
      "lastModified":1334692961000
    },
    {
      "rank":2,
      "bgRating":3020,
      "wins":290,
      "losses":34,
      "played":324,
      "realm":
      {
        "name":"Kel'Thuzad",
  "slug":"kelthuzad",
  "battlegroup":"Nightfall"
      },
      "battlegroup":
      {
        "name":"Nightfall",
  "slug":"nightfall"
      },
      "character":
      {
        "name":"Player2",
  "realm":"Kel'Thuzad",
  "class":3,
  "race":2,
  "gender":1,
  "level":85,
  "achievementPoints":6650,
  "thumbnail":"kelthuzad/173/xxx-avatar.jpg"
      },
      "lastModified":1334692961000
    },
    ...
  ]
}
```

### Quest Resources

Quest APIs currently provide quest information.

#### Quest API

The quest API provides detailed quest information.

```plain
URL = Host + "/api/wow/quest/" + QuestId
```

*An example quest API request and response*
```plain
GET /api/wow/quest/13146 HTTP/1.1
Host: us.battle.net
```
```json
{
  "id":13157,
  "title":"The Crusaders' Pinnacle",
  "reqLevel":77,
  "suggestedPartyMembers":0,
  "category":"Icecrown",
  "level":79
}
```

### Data Resources

The data APIs provide information that can compliment profile
information to provide structure, definition and context.

#### Battlegroups

The battlegroups data API provides the list of battlegroups for this
region. Please note the trailing / on this request url.

```plain
URL = Host + "/api/wow/data/battlegroups/"
```

#### Character Races

The character races data API provides a list of character races.

```plain
URL = Host + "/api/wow/data/character/races"
```

#### Character Classes

The character classes data API provides a list of character classes.

```plain
URL = Host + "/api/wow/data/character/classes"
```

#### Character Achievements

The character achievements data API provides a list of all of the
achievements that characters can earn as well as the category
structure and hierarchy.

```plain
URL = Host + "/api/wow/data/character/achievements"
```

#### Guild Rewards

The guild rewards data API provides a list of all guild rewards.

```plain
URL = Host + "/api/wow/data/guild/rewards"
```

#### Guild Perks

The guild perks data API provides a list of all guild perks.

```plain
URL = Host + "/api/wow/data/guild/perks"
```

#### Guild Achievements

The guild achievements data API provides a list of all of the
achievements that guilds can earn as well as the category structure
and hierarchy.

```plain
URL = Host + "/api/wow/data/guild/achievements"
```

#### Item Classes

The item classes data API provides a list of item classes.

```plain
URL = Host + "/api/wow/data/item/classes"
```

## API Policy

With the continued popularity of third-party applications, which are
referred to hereafter as "applications" or "web applications", created
by the community of players for use with our games, Blizzard
Entertainment has created formal guidelines for their design and
distribution. These guidelines have been put in place to ensure the
integrity of our games and to help promote an enjoyable gaming
environment for all of our players.

Thank you for reading these guidelines, and for helping Blizzard
Entertainment continue to deliver high-quality gameplay experiences.

### Intended Audience

The Third-Party API Usage Policy is for developers developing
applications, where applications include distributed and
non-distributed products and services that at any point engage
Blizzard Entertainment Web API resources. Web API resources include
any data that can be accessed through HTTP requests to URLs on the
Battle.net website that begin with "/api".

Example applications include, but are not limited to:

* Client libraries
* Desktop applications
* Services and deamons such as websites and web services
* Scripts and non-compiled applications and utilities

Blizzard Entertainment reserves the right to change the location and
definition of what constitutes an application and Blizzard
Entertainment Web API resources at any time.

### Service Availability Notice

Blizzard Entertainment makes no guarantee of the availability of any
data, functionality, or feature provided by or through the API. In
addition, Blizzard Entertainment may at any time revoke access to the
API or disable part or all of the API without any warning or notice.

Applications must abide by the following access guidelines.

The following guidelines have been put in place to ensure that all
users of an API will be able to access it:

* Applications may make up to a total of 10,000 unauthenticated
  requests per day.
* Applications may make up to a total of 50,000 authenticated requests
  per day.
* Applications may not use multiple forms of access, including making
  any combination of unauthenticated and authenticated requests or
  using multiple API keys, to make more requests than permitted by the
  guidelines above. Applications may not use other third-party
  services to make additional requests on their behalf.
* Applications may not sell, share, transfer, or distribute
  application access keys or tokens.

Applications may be classified, solely at Blizzard Entertainment's
discretion, to allow fewer or greater numbers of requests per
day. Blizzard Entertainment also reserves the right to revoke access
to the API completely and without warning.

### Applications may not charge premiums for features that use the
    API.

"Premium" versions of applications offering additional for-pay
features are not permitted, nor can players be charged money to
download an application, charged for services related to the
application, or otherwise be required to offer some form of monetary
compensation to download or access an application when those features
use the API. Applications may not include interstitials soliciting
donations before features or functionality becomes available to the
player.

### Applications must not negatively impact Blizzard Entertainment
    games, services, or other players.

Applications must perform no function which, in Blizzard
Entertainment's sole discretion, negatively impacts the performance of
Blizzard Entertainment games or services, or otherwise negatively
affects the game for other players.

### Application code must be completely visible.

The programming code of an application must in no way be hidden or
obfuscated, and must be freely accessible to and viewable by the
general public.

### Applications may not imply any association with Blizzard
    Entertainment.

Applications may not imply any association with, or endorsement by,
Blizzard Entertainment.

### Applications must not contain offensive or objectionable material.

Blizzard Entertainment requires that applications contain no
offensive, obscene, or otherwise objectionable material, as determined
by Blizzard's sole discretion. Applications should contain only
content appropriate for the ESRB rating for the related game(s). For
example, World of Warcraft has been rated "T for Teen" by the ESRB,
and has received similar ratings from other ratings boards around the
world.

### Blizzard trademarks, titles, or tradenames should not be used when
    naming an application.

Applications may not use names based on Blizzard's trademarks or taken
from Blizzard's products as the name, or part of the name, of the
application.

### License for Use

Applications that use Blizzard Entertainment intellectual property,
such as Blizzard Web API resources, require a license for that
use. Blizzard Entertainment may at its sole discretion request that
any application that uses its intellectual property be removed and no
longer distributed.

### Policy Compliance Notice

Blizzard Entertainment is committed to maintaining the integrity of
our games and services and to providing safe, fair, and fun gaming
environment for all of our players. As such, failure to abide by the
guidelines in this policy may result in measures up to and including
legal action, when necessary.
