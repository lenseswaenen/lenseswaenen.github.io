---
layout: post
title: "Table tennis home advantage analysis"
subtitle: "And a festive bonus!"
date: 2021-07-21
background: /img/posts/tabletennis/lisa-keffer-i0kB5B9J8Ds-unsplash.jpg
---

# English summary
As the most likely audience for this post are Flemish table tennis players, most of it is written in Dutch. Here a short summary:

I have played table tennis competitively for the last 15 years at table tennis club TTK Minderhout. Unfortunately a lot of visiting players do not like our accommodation and believe we have an unfair home advantage. All Flemish table tennis matches since season 2006-2007 are accessible through an open API. Analysing that data, shows that the average home advantage across all Antwerp clubs is about 5 percentage points (difference in win probability of home games versus out games). TTK Minderhout has no excessive home advantage whatsoever.

# Heeft TTK Minderhout een uitzonderlijk thuisvoordeel?
Nee, is het antwoord.

Ik speel sinds 2002 tafeltennis bij tafeltennisclub TTK Minderhout. Na 4 jaar jeugdcompetitie, heb ik er nu ook een 15-tal jaren herencompetitie binnen de provincie Antwerpen opzitten, op een korte uitzwerving naar Ekerse na. TTK Minderhout huurt al zolang ik er speel tafeltennislokaal de noodkerk van het kerkfabriek. De speelzaal heeft een betonnen vloer, wat enige voordelen heeft inzake multifunctioneel gebruik van de zaal.

De meeste Antwerpse clubs spelen op een houten vloer of een sportvloer als in een turnzaal. In tafeltennis beinvloedt de ondergrond best wel de speelervaring, zoals in tennis gravel en gras dat ook doen (doch minder extreem in tafeltennis). Daar bezoekende ploegen en spelers onze vloer niet gewend zijn, en tafeltennisspelers een ras apart zijn, geeft dit regelmatig klagende en jammerende tegenstanders, wat voor ons als gastploeg al jaren een domper op de speelvreugde zet. Maar goed, misschien hebben ze gelijk. Vandaar dat ik tijdens de competitiestilte tijdens de coronacrisis eens de moeite heb genomen om na te rekenen wat de statistieken zeggen.

# TabT API

Sinds seizoen 2006-2007 worden alle wedstrijden van de Vlaamse Tafeltennis Liga (VTTL) bijgehouden op <https://competitie.vttl.be/>. In de beginjaren was de URL iets als ```vttl.frenoy.net``` (URL werkt niet meer), naar de hoofdontwikkelaar Gaetan Frenoy. Tafeltennis was daarmee een van de eersten om zo uitgebreid digitaal te gaan. Badminton, squash, tennis zijn veel later gevolgd (badminton en squash met een toernooi.nl systeem, tennisvlaanderen met zijn eigen systeem). Na al die tijd vind ik het VTTL systeem het meest responsieve en gebruiksvriendelijke systeem, althans om snel en veel gedetailleerde resultaten te kunnen raadplegen. Ook de Franstalige vleugel en recreatieve federatie Sporcrea gebruiken ditzelfde systeem.

Gaetan Frenoy is zelfs zo goed geweest om een publieke SOAP API naar de database met spelers en uitslagen te voorzien. Deze heet de TabT API: <http://tabt.frenoy.net/index.php?l=EN&display=MainPage_EN>

Nu ben ik zelf geen held in web development en het ontrafelen van dergelijke APIs. Volgende website van TTC Erembodegem is voor mij daarom erg waardevol geweest om geautomatiseerd de nodige wedstrijd data te downloaden: Op <http://ttc-erembodegem.be/tabtapi-test/> kan je bepaalde queries via een formulier doen. De broncode achter dit formulier is ook openbaar, namelijk: <https://github.com/Laoujin/ttc-test-tabtapi>

Om de SOAP API aan te roepen, is de ```zeep``` module een handige manier


```python
import zeep

wsdl = 'https://api.vttl.be/?wsdl'
client = zeep.Client(wsdl=wsdl)
```


```python
credentials = {'Account':'foo', 'Password':'bar'}
```


```python
client.service.Test(credentials)
```




    {
        'Timestamp': datetime.datetime(2021, 7, 21, 14, 12, 38, tzinfo=<FixedOffset '+02:00'>),
        'ApiVersion': '0.7.25',
        'IsValidAccount': False,
        'Language': 'nl',
        'Database': 'vttl',
        'RequestorIp': '2.15.139.88',
        'ConsumedTicks': 18,
        'CurrentQuota': 0,
        'AllowedQuota': 8000,
        'PhpVersion': None,
        'DbVersion': None
    }



Wat je nog meer kan met niet-dummy credentials, weet ik niet. De response geeft ook wat aan van quota. Als we hier tegenaanlopen, dan doen we even een ```sleep``` en proberen we nog eens. Excuses Gaetan!


```python
result = client.service.GetClubs()
print(result.ClubCount)
```

    534
    

Een ```print``` van de hele result is zeer leerrijk, maar laat ik hier achterwege omwille van de lengte. Dit resultaat ziet er erg uit als een ```json``` dingetje. In praktijk is het een ```zeep``` object, dat eerder als een klasse met attributen aangeroepen moet worden dan een ```dict``` met keys.


```python
result = client.service.GetSeasons()
print(result.CurrentSeason, result.CurrentSeasonName)
```

    22 2021-2022
    

Seizoenen hebben naast een naam ook een integer label, wat gebruikt kan worden als argument van API calls (bvb. leden van een club in een bepaald seizoen).

Voor clubs is dan weer de clubcode als string nodig. Voor TTK Minderhout is deze ```A135```


```python
result = client.service.GetClubTeams(credentials, 'A135')
result.ClubName
```




    'TTK Minderhout'



# Thuisvoordeel TTK Minderhout

Als we ```GetMatches``` aanklikken op de TTC Erembodgemse test website, krijgen we signatuur van de ```GetMatches``` functie te zien. Deze ziet er zo uit:
Request structure:

	Array
	(
		[Credentials] => Credentials Object
			(
				[Account] => 
				[Password] => 
			)

		[DivisionId] => 
		[Club] => 
		[Team] => 
		[DivisionCategory] => 
		[Season] => 
		[WeekName] => 
		[Level] => 
		[ShowDivisionName] => no
		[WithDetails] => 
		[MatchId] => 
	)
	
Met deze API call trekken we alle competitieve ontmoetingen van TTK Minderhout in seizoen 2017-2018 binnen. Dit waren er 130. 


```python
club = 'A135'
result = client.service.GetMatches(credentials, None, club, None, None, 18)
print(result.MatchCount)
```

    130
    

Een enkele ontmoeting ziet er dan zo uit:


```python
result.TeamMatchesEntries[0]
```




    {
        'DivisionName': None,
        'MatchId': 'PANTH01/014',
        'WeekName': '01',
        'Date': datetime.date(2017, 9, 15),
        'Time': datetime.time(20, 0),
        'Venue': 1,
        'VenueClub': 'A135',
        'VenueEntry': {
            'Id': None,
            'ClubVenue': None,
            'Name': 'TTC Minderhout (Noodkerk)',
            'Street': 'Schoolstraat',
            'Town': '2322 Minderhout',
            'Phone': None,
            'Comment': None
        },
        'HomeClub': 'A135',
        'HomeTeam': 'Minderhout A',
        'AwayClub': 'A139',
        'AwayTeam': 'Hallaar A',
        'Score': '13-3',
        'MatchUniqueId': 278412,
        'NextWeekName': '02',
        'PreviousWeekName': None,
        'IsHomeForfeited': False,
        'IsAwayForfeited': False,
        'MatchDetails': None,
        'DivisionId': 3324,
        'DivisionCategory': 1,
        'IsHomeWithdrawn': 'N',
        'IsAwayWithdrawn': 'N',
        'IsValidated': True,
        'IsLocked': True
    }



Mijn houtje touwtje code hieronder parst de ```Score``` string en filtert bovendien de jeugdwedstrijden eruit op basis van de ```MatchID``` string.


```python
home_games = 0
away_games = 0
home_wins = 0
home_loss = 0
away_wins = 0
away_loss = 0
for i, entry in enumerate(result.TeamMatchesEntries):
    if 'J' in entry.MatchId: # Filter jeugdwedstrijden er uit...
        continue
        
    if not entry.Score:
        continue
        
    home, away = entry.Score.split('-')
    try:
        home = int(home)
        away = int(away)
    except:
        continue
        
    if entry.HomeClub == 'A135':
        home_games += 1
        home_wins += home
        home_loss += away
        
    else:
        away_games += 1
        away_wins += away
        away_loss += home

print(home_games, away_games)
```

    53 53
    

We tellen evenveel uitwedstrijden als thuiswedstrijden


```python
print("%i, %i, %.3f, %i, %i, %.3f" % (home_wins, home_loss, home_wins/(home_wins + home_loss), away_wins, away_loss, away_wins/(away_wins + away_loss)))
```

    476, 371, 0.562, 426, 422, 0.502
    
In dit ene seizoen heeft TTK Minderhout 476 individuele overwinningen thuis gehad, tegenover 371 nederlagen, goed voor een winstpercentage van 56%. Op verplaatsing heeft TTK Minderhout 426 overwinningen geboekt tegenover 422 nederlagen, goed voor een winstpercentage van 50%. 

```python
home_wins/(home_wins + home_loss) - away_wins/(away_wins + away_loss)
```


    0.05962498050834242



In dit specifieke seizoen heeft TTK Minderhout een thuisvoordeel van 6 procentpunten gekend. Bovendien geen slecht seizoen voor de club met meer gewonnen dan verloren wedstrijden.

Laten we nu itereren over alle beschikbaren seizoenen:


```python
divisionId = None
club = 'A135'
team = None
divisionCategory = None
for season in range(7, 22):
    result = client.service.GetMatches(credentials, divisionId, club, team, divisionCategory, season)

    home_games = 0
    away_games = 0
    home_wins = 0
    home_loss = 0
    away_wins = 0
    away_loss = 0
    for i, entry in enumerate(result.TeamMatchesEntries):
        if 'J' in entry.MatchId: # Jeugd...
            continue

        if not entry.Score:
            continue

        home, away = entry.Score.split('-')
        try:
            home = int(home)
            away = int(away)
        except:
            continue

        if entry.HomeClub == 'A135':
            home_games += 1
            home_wins += home
            home_loss += away

        else:
            away_games += 1
            away_wins += away
            away_loss += home
        #print(i, home, away)
    
    try:
        home_ratio = home_wins/(home_wins + home_loss)
        away_ratio = away_wins/(away_wins + away_loss)
    except:
        home_ratio = -1
        away_ratio = -1
        
    print("%i, %i, %i -- %i, %i, %.3f, %i, %i, %.3f, \t %.3f" % (season, home_games, away_games,
                                                        home_wins, home_loss, home_ratio, 
                                                        away_wins, away_loss, away_ratio, home_ratio - away_ratio))
```

    7, 33, 33 -- 304, 200, 0.603, 319, 184, 0.634, 	 -0.031
    8, 42, 42 -- 337, 335, 0.501, 284, 388, 0.423, 	 0.079
    9, 60, 60 -- 683, 277, 0.711, 627, 333, 0.653, 	 0.058
    10, 61, 61 -- 500, 474, 0.513, 446, 529, 0.457, 	 0.056
    11, 58, 55 -- 476, 452, 0.513, 418, 462, 0.475, 	 0.038
    12, 53, 51 -- 285, 563, 0.336, 212, 604, 0.260, 	 0.076
    13, 42, 40 -- 282, 389, 0.420, 233, 407, 0.364, 	 0.056
    14, 41, 41 -- 393, 263, 0.599, 347, 309, 0.529, 	 0.070
    15, 41, 41 -- 367, 289, 0.559, 329, 327, 0.502, 	 0.058
    16, 28, 29 -- 261, 187, 0.583, 273, 191, 0.588, 	 -0.006
    17, 32, 32 -- 296, 216, 0.578, 273, 239, 0.533, 	 0.045
    18, 53, 53 -- 476, 371, 0.562, 426, 422, 0.502, 	 0.060
    19, 60, 60 -- 604, 356, 0.629, 536, 424, 0.558, 	 0.071
    20, 52, 52 -- 409, 423, 0.492, 342, 489, 0.412, 	 0.080
    21, 11, 12 -- 81, 95, 0.460, 82, 110, 0.427, 	 0.033
    

Het schommelt dus rond de 3 en 8 procentpunten, met een gemiddelde van 5.3.

# Thuisvoordeel van alle Antwerpse clubs

De code hieronder genereert de lijst van alle Antwerpse clubs.


```python
names = []
for i, club in enumerate(client.service.GetClubs().ClubEntries):
    if club.CategoryName != 'Antwerpen':
        continue
    
    if club.VenueCount == 0:
        continue
    
    clubId = club.UniqueIndex
    names.append(club.Name)
    
    print(i, clubId, club.Name)
```

    1 A003 Salamander
    2 A008 Brasgata
    3 A062 AFP Antwerpen
    4 A074 Hove
    5 A075 Rupel
    6 A095 Turnhout
    7 A097 Nijlen
    8 A105 Dessel
    9 A115 Dylan Berlaar
    10 A117 Geelse
    11 A118 Rijkevorsel
    12 A123 Virtus
    13 A127 Retie
    14 A129 Borsbeek
    15 A130 Schoten
    16 A135 Minderhout
    17 A136 Zoersel
    18 A138 Blue Rackets
    19 A139 Hallaar
    20 A141 Merksplas
    21 A142 Tecemo
    22 A147 Gierle
    23 A155 Wommelgem
    24 A159 Real
    25 A160 Walem
    26 A167 Lille
    27 A176 Sokah
    28 A182 Nodo
    29 A186 Willebroek
    30 A201 Hulshout
    31 A211 Zwijndrecht
    32 A212 Antonius
    33 A216 Henricus
    34 A218 Poppel
    35 A219 Stari Bog
    

We hebben dus een drievoudige lus nodig: over alle clubs, alle seizoenen en dan alle gevonden wedstrijden: 

```python
import time

for i, club in enumerate(client.service.GetClubs().ClubEntries):
    if club.CategoryName != 'Antwerpen':
        continue
    
    if club.VenueCount == 0:
        continue
        
    clubId = club.UniqueIndex
    
    time.sleep(1.) # Sleep helpt met de quota niet te overschrijven / server niet te overbelasten
    
    divisionId = None
    team = None
    divisionCategory = None

    home_games = 0
    away_games = 0
    home_wins = 0
    home_loss = 0
    away_wins = 0
    away_loss = 0

    for season in range(15, 22):
        success = False
        while not success:
            try:
                result = client.service.GetMatches(credentials, divisionId, clubId, team, divisionCategory, season)
                success = True
            except Exception as e:
                print('fail', e)
                time.sleep(30.) # Sorry Gaetan!


        for i, entry in enumerate(result.TeamMatchesEntries):
            if 'J' in entry.MatchId: # Jeugd...
                continue

            if not entry.Score:
                continue

            home, away = entry.Score.split('-')
            try:
                home = int(home)
                away = int(away)
            except:
                continue

            if entry.HomeClub == clubId:
                home_games += 1
                home_wins += home
                home_loss += away

            else:
                away_games += 1
                away_wins += away
                away_loss += home
            #print(i, home, away)

    try:
        home_ratio = home_wins/(home_wins + home_loss)
        away_ratio = away_wins/(away_wins + away_loss)
    except:
        home_ratio = -1
        away_ratio = -1

    print("%4s, %20s -- %5i, %5i -- %5i, %5i, %.3f, %5i, %5i, %.3f, \t %.3f" % (clubId, club.Name, home_games, away_games,
                                                        home_wins, home_loss, home_ratio, 
                                                        away_wins, away_loss, away_ratio, home_ratio - away_ratio))
```

De volledige resultaten staan hieronder. De laatste kolom geeft het thuisvoordeel aan:


```python
Full results:
A003,           Salamander --  1602,  1587 -- 14166, 11005, 0.563, 12523, 12440, 0.502, 	 0.061
A008,             Brasgata --  1478,  1458 -- 11294, 11959, 0.486, 10144, 12836, 0.441, 	 0.044
A062,        AFP Antwerpen --  2082,  2070 -- 16127, 15710, 0.507, 14521, 17122, 0.459, 	 0.048
A074,                 Hove --   660,   664 --  5976,  4522, 0.569,  5454,  5108, 0.516, 	 0.053
A075,                Rupel --   767,   762 --  6780,  5491, 0.553,  6121,  6071, 0.502, 	 0.050
A095,             Turnhout --   740,   716 --  6239,  5350, 0.538,  5506,  5699, 0.491, 	 0.047
A097,               Nijlen --   760,   762 --  5381,  6704, 0.445,  5175,  6956, 0.427, 	 0.019
A105,               Dessel --   150,   152 --  1084,  1217, 0.471,  1010,  1322, 0.433, 	 0.038
A115,        Dylan Berlaar --  1589,  1566 -- 12441, 11110, 0.528, 11000, 12213, 0.474, 	 0.054
A117,               Geelse --  1473,  1491 -- 12544, 10559, 0.543, 11367, 12036, 0.486, 	 0.057
A118,          Rijkevorsel --   391,   393 --  3608,  2648, 0.577,  3108,  3180, 0.494, 	 0.082
A123,               Virtus --   478,   471 --  3660,  3597, 0.504,  3150,  4006, 0.440, 	 0.064
A127,                Retie --   667,   662 --  5641,  4987, 0.531,  5070,  5476, 0.481, 	 0.050
A129,             Borsbeek --   589,   580 --  4814,  4609, 0.511,  4276,  5001, 0.461, 	 0.050
A130,              Schoten --  1282,  1274 -- 10391, 10120, 0.507,  9411, 10972, 0.462, 	 0.045
A135,           Minderhout --   667,   662 --  5754,  4890, 0.541,  5147,  5418, 0.487, 	 0.053
A136,              Zoersel --   955,   951 --  8436,  6485, 0.565,  7446,  7401, 0.502, 	 0.064
A138,         Blue Rackets --   540,   543 --  4201,  4347, 0.491,  3873,  4725, 0.450, 	 0.041
A139,              Hallaar --   840,   851 --  6883,  6457, 0.516,  6228,  7296, 0.461, 	 0.055
A141,            Merksplas --   460,   461 --  4167,  2663, 0.610,  3917,  2952, 0.570, 	 0.040
A142,               Tecemo --   687,   673 --  5844,  4913, 0.543,  5221,  5320, 0.495, 	 0.048
A147,               Gierle --  2094,  2034 -- 16930, 16306, 0.509, 14774, 17510, 0.458, 	 0.052
A155,            Wommelgem --   350,   354 --  3056,  2543, 0.546,  2707,  2955, 0.478, 	 0.068
A159,                 Real --    24,    21 --    44,    76, 0.367,    53,    52, 0.505, 	 -0.138
A160,                Walem --   421,   420 --  3320,  3416, 0.493,  3022,  3697, 0.450, 	 0.043
A167,                Lille --   457,   448 --  3642,  3643, 0.500,  3148,  4009, 0.440, 	 0.060
A176,                Sokah --  2077,  2082 -- 17004, 13898, 0.550, 15337, 15694, 0.494, 	 0.056
A182,                 Nodo --  1936,  1921 -- 14876, 13391, 0.526, 13485, 14640, 0.479, 	 0.047
A186,           Willebroek --   484,   485 --  4022,  3722, 0.519,  3390,  4370, 0.437, 	 0.083
A201,             Hulshout --   349,   354 --  3016,  2538, 0.543,  2954,  2679, 0.524, 	 0.019
A211,          Zwijndrecht --   407,   407 --  3801,  2645, 0.590,  3473,  2973, 0.539, 	 0.051
A212,             Antonius --   917,   920 --  7923,  6221, 0.560,  7271,  6968, 0.511, 	 0.050
A216,             Henricus --    94,    95 --   957,   547, 0.636,   858,   662, 0.564, 	 0.072
```

Merk op dat Real een nieuwe club is, waardoor zij nog maar weinig statistiek hebben en op een extreme waarde van -14 procentpunten uitkomen. In de visualisatie verderop negeren we hen. Andere kolommen waar we niet dieper op ingaan geven winstpercentages. We zien daar bijvoorbeeld hoe TTK Merkplas, die de laatste jaren in opmars is, met een percentage van 60 procent, bij de top hoort.

Wat sorteren en visualiseren:

```python
home_advantage = [0.061,0.044,0.048,0.053,0.050,0.047,0.019,0.038,0.054,0.057,0.082,0.064,0.050,0.050,0.045,0.053,0.064,0.041,0.055,0.040,0.048,0.052,0.068,-0.138,0.043,0.060,0.056,0.047,0.083,0.019,0.051,0.050,0.072]

import numpy as np

names = np.array(names)
home_advantage = np.array(home_advantage)

inds = np.argsort(home_advantage)
```


```python
%matplotlib notebook

import matplotlib.pyplot as plt

plt.figure(figsize=(6, 10))
plt.barh(np.arange(len(inds)-1), home_advantage[inds[1:]])
plt.barh([18], home_advantage[inds[18+1]], color='C1')
ax = plt.gca()
ax.set_yticks(np.arange(len(inds)-1))
ax.set_yticklabels(names[inds[1:]])
plt.title('Thuisvoordeel in procentpunten')
plt.tight_layout()
```


    <IPython.core.display.Javascript object>



<img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAu4AAATiCAYAAADlD2aLAAAgAElEQVR4XuydCbiVUxv37yLJ1ISKqAglY4VSqEhIgylDpSIy5TUkQ1JRvBkzNYgyJEMkcxGVokivpDlDoaLJLFPOd/3X9z3722e3zzn7OefZZ+/n2b91XV109nrWutfvXof/up973btMXl5entEgAAEIQAACEIAABCAAgawmUAbhntX+wTgIQAACEIAABCAAAQg4Agh3NgIEIAABCEAAAhCAAARCQADhHgInYSIEIAABCEAAAhCAAAQQ7uwBCEAAAhCAAAQgAAEIhIAAwj0ETsJECEAAAhCAAAQgAAEIINzZAxCAAAQgAAEIQAACEAgBAYR7CJyEiRCAAAQgAAEIQAACEEC4swcgAAEIQAACEIAABCAQAgII9xA4CRMhAAEIQAACEIAABCCAcGcPQAACEIAABCAAAQhAIAQEEO4hcBImQgACEIAABCAAAQhAAOHOHoAABCAAAQhAAAIQgEAICCDcQ+AkTIQABCAAAQhAAAIQgADCnT0AAQhAAAIQgAAEIACBEBBAuIfASZgIAQhAAAIQgAAEIAABhDt7AAIQgAAEIAABCEAAAiEggHAPgZMwEQIQgAAEIAABCEAAAgh39gAEIJBWAmXKlPE1fq1atWzlypU2ffp0a9mypXXr1s0ef/xxX2P47Vy7dm1btWqV5eXl+X000v27d+9uTzzxhE2bNs1atGhRqmuVz3v06GEDBgywgQMHpjQ3fkwJE50gAIEQE0C4h9h5mA6BMBCQ+Etss2bNsi+++MIOPfRQO+yww/J9vOuuu9rdd9+NcM8C5yLcs8AJITNBB3Xv8B0y012AwO9hMWxrxN7wE0C4h9+HrAACoSPgCcLCoqmlGXHXIeLvv/+2evXqhY5lOg0Om3DHj+ncDamNjXBPjRO9IFBcAgj34pLjOQhAoNgEsk24F3shEX8wbMI94u4IxfIQ7qFwE0aGmADCPcTOw3QIhJWAX+F+7733Wr9+/WzSpEm2adMm22+//eyaa66xCy64IB+CoqL0BQnRgnKjlyxZYrfffrt98MEHtnr1att5551tzz33dPne119/vdWoUcNefPFFO/PMM+3ss8+2Z599NqlLLrvsMhsxYoQ98sgjdtFFF8X6fPPNNzZ48GCbPHmyfffdd1axYkVr3ry53XjjjXbEEUckHWv27Nn23//+19n0888/OxtOOeUUu/nmm22PPfbI90z8q//zzjvPbrnlFpevvn79eps4caJ17NjR9dcahg4dap999pntsssu1qZNGzfHTTfdVGCOu8bQM6+++qq7H1ChQgVr0qSJs/3YY49NarvG17jy04YNG6xq1apuLr15kQ/iW3HSFpL5Ufcl6tSpY8cdd5y9+eabNmjQIHvmmWcc77322sv5o2/fvpbqXYxEplqv1vPHH3+4tC/tU/kjvsXb8MorrzgbxP/bb7+1yy+/3IYNG+a6L1682IYMGWLvvvuubdy40XbbbTc7/vjj3ZgHHHBAUqZ65q677nLPaE2VKlVyfc844wz7z3/+k++ZX3/91fS7NGHCBJeqtu2229rhhx9uV199dWwveA/45eZxSWak2IuRmuejf//91x544AEbNWqUffnll24vyOZbb73VrSG+6fdtxowZ9tVXX221T+Lt9ObQs7oXIc5jx461Ro0aOYYzZ860v/76yxo3bmx33HGHHX300bFpvDmS2a8x4lP+/Oxjv3aE9b/p2F16BBDupceamSAAgf9HwI9w79Chg0lA//TTT3bkkUeaxMd7771nW7ZssdGjR1vPnj1jXIMU7v/73/+ciJYg07wSf7/88osTGbLHu7D5559/WrVq1ZwgWLdune200075/PzPP/84cS2RLWFVuXJl97n+59+qVSsnYJWiI9H39ddfO0EuQTV+/Hg766yz8o01btw4JyAkeiQ6JDxl5/Lly50NWn98uo8nps455xx74403nDjSWn744Qe78sorrW3btvbQQw9Z7969bZtttnHiVncMJHBkg+4gvPbaa1tdTl26dKmdcMIJ7jCz77772iGHHOKE5pw5c0zrfeqpp0wHhfimw4F+Jk4SUuIp8fjJJ584uyTMGjRoEHskaOHetGlTt8ZFixY5BmqaU/6VqNMBKpXm2dWlSxd3aKlSpYodddRRtmbNGsdNbcyYMfmEnicuNa/Wr4OOWJctW9ax08HlnXfesXbt2tnmzZutYcOGtv/++5s4z58/3+0p+e+YY47JZ6IEeNeuXU17UOwOPvhgd7BduHChsyf+svX333/v9puEvg6f8sHvv/9uOgj+9ttvTsjecMMNsfE9m1Plpnsrjz76qDvo7bjjju4w6zXtSW9sT7jrwKKDrASzfifkC9koHhpLh2Sv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V9+jRw97/fXX3RuBJ5980tasWWPz5s0zHVKUJ9+vXz9799137ddff7X99tvPLrnkErv88su3Qrpo0SKXWz9t2jT7+eefXZ7+RRddZFdffbWVLVs21j+ZcNd8J598su299942efJk9+Yi1UbEPVVS9IMABCAAAQhAoKQEEO4lJViKz0sEn3nmmTZq1Ci7+OKL3cwS03fddZetX7/eiWOJTwlgNYnh5s2bO0F7yy23FFu46wCgsSXGL7zwQjv44IPt66+/tocffth23nlnk3j1xK4n3A899FDbZZddnL2rV692B44qVarYggUL3D/VPOF+0EEHubcBnTt3tjJlyrhn9CbhyCOPdP+84oorrEaNGvbSSy85YX7NNdfYPffcEyM/Z84cl8evA0S3bt2scuXKrp/y/S+99FIbPnx4gcJdl3V1d0BpRK+88opbj5+GcPdDi74QgAAEIAABCJSEAMK9JPRK+VmJ8913390J3HHjxrnZ27Zt60TvpEmTrH///vbUU0/FqsboQupNN93kRKwqxxQ34q6DgcZRHr2i916bP3++NW7c2K677jp3+VXNE+6K/s+YMcO23XZb93Pvcm3fvn3dYSNeuB944IEuyr799tvHxj777LNtwoQJ9sEHH1iTJk3cz//9919r166dvfnmm7Z48WL31iEvL88OOeQQK1++vDuo6J9e69Onj917772xvvp5fMT95ZdftnPOOcfatGljzz33XL5nk7lWaUqJdwj09qNLly5WvdswK189+6rolPIWZToIQAACEIAABNJIAOGeRrjpGFoiV2kjinhLyCp6LcF+7bXX2ltvveVE6KpVq1zkXekfSjP56aefnCgurnBv2LChyzl/7bXXtlrSMcccYzvttJPNnTs3n3AfP368nXvuufn6K/deQl6iO16433fffXbVVVfF+uoCrVJxjjrqKJfiE990eDjuuOOc+NchQBF8RfeHDRvmDjTx7dNPP3WR+IceeiiWXuMJd6XR6O1B165d7dFHH7VtttmmSHd5h5JkHRHuReKjAwQgAAEIQAACJSSAcC8hwNJ+XDneSpX56quv7IcffjCJ6g8//NCllfzyyy8uTUQpKBLNEvWKRs+cOdOZWVzhrio0mzdvLnCpOiTosKDmiVtF0GVbfFO0fOrUqbGxvFQZpajoM6999913LjVGKTJKsYlvGzZssN12283l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Cy+8kM9Pxx9/vL333nu266672tq1a/N9NmDAALv11lvt/ffft6OPPjqs/i3SboR7kYjoAAEIQAACEMhaAgj3rHVNWgyLvHAXtcMOO8wJ8++//z4G8e+//7ZKlSq5aPsTTzxhy5Yts/333z/2ecuWLe2jjz4yCf9y5cqlBX42DIpwzwYvYAMEIAABCECgeAQQ7sXjFtanckK4K0XmwQcftCVLlli9evWcr2bPnu0i6Z988on75wMPPGA9e/Z0n/31119O1Ddt2tSUMvPzzz+7lJoXX3zRHQBq1KhhZ555pvvZzjvvHPO9/q70m/nz59vo0aNd+s0ff/xhJ510kj3yyCOur6L4Y8aMsQ0bNtgxxxxjjz32mO21116BjqHBNm7caLfddptNmjTJ1qxZY7vttpt76zB48GC3Nq8VJNyVWiQmq1evtn322ceuv/56++qrr9z68vLy8u33RYsWuZ9PmzbNsapTp45ddNFFdvXVV1vZsmVjfWvXrm01a9Z04+qzuXPnurchGvuKK65w48tXM2bMsPLly1uvXr3cGsqUKeP794scd9/IeAACEIAABEJIAOEeQqeVwOScEO4S3BLao0aNsosvvtjhGjp0qN111122fv16U3R97733tieffNJ9pvSY5s2bOzGq/Hb9u0Rmjx49rHHjxvbxxx/b2LFjrUmTJi7VxovIe8L98MMPt+rVq1vbtm1t4cKFNnLkSDv99NNt9913N4lcRfm//vpru+++++zYY4+1d999dyvhXpIxfvjhBzvqqKPc4UDrlfBeunSpjRgxwurXr+8OLRLGasmE++233279+vWzZs2aOVs3bdpkDz/8sGOkg068cJ8zZ467CyBB3q1bN6tcubIT8Dq0XHrppTZ8+PB8wl1CfvPmzdalSxeTkNfbDrEVT/E79dRTrUGDBjZhwgQ3jj4///zzfW9xhLtvZDwAAQhAAAIhJIBwD6HTSmByTgh3iXOJ5s6dO9u4ceMcLolqCW5FpPv3729PPfVUrLKKLqTedNNNTjgqSn/ZZZfZf//7XxcZ9pqEv0S9RLkiw2qecFdke+LEibG+Z5xxhr300ksugi+hv80227jPevfubQ899JAtX77c9ttvv8DGuPzyy9065dx99903Zsdrr71m7dq1y2dzonBXpH7PPfe0hg0bOlu33XZb9/zixYvtkEMOsS1btsSEuwS8fqZDgA473mFA/fv06WP33nuve857yyGhvmrVKse8Q4cOblwdLiT69ZZDhwOJfTW9qdCbCKUvaezCmt6CJN5RkN90OKjebZiVr163BL8iPAoBCEAAAhDIXgII9+z1TTosywnhLnAHHnig/frrry7S/e+//1qVKlWcYL/22mvtrbfesjZt2jhRqajyySef7KLgP/30k0svmTlzpovMV6hQIeYDRY11qfW4446zN954I5/onjJlip144omxvnfffbddd911LkVGUXuvPfvss3buuee65zVnvPgv7hgS00qLadWqVb5otzenxLPmUkRbLVG4ezaNHz/e2RbfdNiRrV7EfcGCBXbooYfasGHD3KEovn366acuEq+DiQ4SappbAl2pO/HtoIMOcncMfvvtN9tuu+1iHynFSBH++LsJyX4JvANTss8Q7un4zwZjQgACEIBAthBAuGeLJ0rHjpwR7pdccolLlVEetVJJFFH+8MMP7cgjj7RffvnFpXg8/vjjTqxK1CuSLMGuaLEi85999tlWHpHgVARa0d140a20lAMOOCDW/9FHH3U531OnTjVVsvHa5MmTnYhWdNwTvp4ILe4Y69ats2rVqhW6e5SeozzyZMJdbxZuvPFGmzdvnmMU35SXLpHuCXelw5x99tmFznXLLbe4lCNPuOt+gFJ14ptSkXRo+uabb/L9/JxzznH3CnSRuLBGxL10/mPBLBCAAAQgkH0EEO7Z55N0WpQzwv2ZZ56x8847z+VMS7grFUYRdS8VpFGjRqY/SntRHrtyvHWRsyjhrui90kHihfuKFSusbt3/n57hCXel3rRo0WIr4a40HaV1BDHGd9995y7PduzYMRbpTtxAupyqNSYT7l6aUCrC3YvOq3SmxHeypouqXrqOdzl11qxZWwn3b7/9dqsvgZJwVwnPf/75x/fvADnuvpHxAAQgAAEIhJAAwj2ETiuByTkj3JWeodztCy+80Al3iXZFwL121VVXmSLgEu7XXHONeakqiogr8q5c7O233z7WXznYXqrM66+/HojoDkK46w2AKrXo4qzWU1QrSaqMxL0OAKl+SRXCvShv8DkEIAABCEDAHwGEuz9eYe+dM8JdjlIUXFVNJNp14VSRYq8psqsKKioNqfrtEvc77bSTu8ipC5OqQKMLl17T3/v27ZuvUo2X5pLJiLvs0+FD5Sh1MFGue3yTsNf6lQ6klijcvcuiqVxO1dsGVYARK+W0J6boKAVJaUbegQfhHvb/XGA/BCAAAQhkGwGEe7Z5JL325JRwv+CCC1zZQTXVZ48XtboAqRKOasp7V/67mi5TKg1EJSATy0Gq5GKycpCZFu4q36hSjp9//rl17drVpQBJsOvvqnajA4ZXsz5ZOUjVTlduusbo1KmTqwmvso66uKsNE18OUpxat27tBLreZqg6joS8ymBqLtW099KGEO7p/WVmdAhAAAIQyD0CCPfc8nlOCXflt0uoKq9dUecddtghn7dVelCiW5F1RdS9pi8VUnReUXnlfI+EZgAAIABJREFUkEvgKzqvS5fJvoAp08Jddmt9Klmpy526+KmKOBLeqp6jKi+1atVKGnH31qwa8/rSKn0Bk4S37gSowosE/O+//56Pm9Y7ZMgQe/vtt131HUXzJeDbt2/vvljJq8aDcM+t/7iwWghAAAIQSD8BhHv6GWfTDDkl3LMJfBhtUQ141ZxX6cZsb1xOzXYPYR8EIAABCARBAOEeBMXwjIFwD4+vSs1S1aiPr1mviVWzXXnv+ibW+G9DLTWjfE6EcPcJjO4QgAAEIBBKAgj3ULqt2EYj3IuNLroPKiVIlWL07aZKC1KUXTXwlWKknHUvzSabCSDcs9k72AYBCEAAAkERQLgHRTIc4yDcw+GnUrVy0aJF7ptetTl00VV5/Ko/r7r29evXL1VbijsZwr245HgOAhCAAATCRADhHiZvldxWhHvJGTJCFhJAuGehUzAJAhCAAAQCJ4BwDxxpVg+IcM9q92BccQkg3ItLjucgAAEIQCBMBBDuYfJWyW1FuJecISNkIQGEexY6BZMgAAEIQCBwAgj3wJFm9YAI96x2D8YVl0DQG7u4dvAcBCAAAQhAAAIQCIpA0PqmTF7812oGZSXjQMAngaA3ts/p6Q4BCEAAAhCAAAQCJxC0vkG4B+4iBiwOgaA3dnFs4BkIQAACEIAABCAQJIGg9Q3CPUjvMFaxCQS9sYttCA9CAAIQgAAEIACBgAgErW8Q7gE5hmFKRiDojV0ya3gaAhCAAAQgAAEIlJxA0PoG4V5ynzBCAASC3tgBmMQQEIAABCAAAQhAoEQEgtY3CPcSuYOHgyJAOcigSDIOBCAAAQgUhwBlGotDjWeKIoBwL4oQn4eSAMI9lG7DaAhAAAKRIYBwj4wrs2ohCPescgfGBEUA4R4UScaBAAQgAIHiEEC4F4cazxRFAOFeFCE+DyUBhHso3YbREIAABCJDAOEeGVdm1UIQ7lnlDowJigDCPSiSjAMBCEAAAsUhgHAvDjWeKYoAwr0oQnweSgII91C6DaMhAAEIRIYAwj0yrsyqhSDcs8od2WFM7dq1rUWLFvb4449nh0HFsALhXgxoPAIBCEAAAoERQLgHhpKB4ggg3HNgO5xxxhk2adIkW7VqldWsWTPpimfNmmXHHHOM9enTxyZMmOBLuN97771WpUoV6969e9bQRLhnjSswBAIQgEBOEkC456Tb075ohHvaEWd+gldffdXat29vd9xxh91www1JDerVq5c98sgjtmDBAtt3331tm222sfLly6dkvA4DdevWtenTp6fUvzQ6IdxLgzJzQAACEIBAQQQQ7uyNdBBAuKeDapaN+c8//9iee+5pVatWtcWLF29l3Z9//mnVq1e3ffbZx+bNm5eS9Xl5efb777/bjjvu6KL4CPeUsNEJAhCAAARyhADCPUccXcrLRLiXMvBMTXf11VfbsGHDbO7cuda4ceN8Zig1plOnTnb//ffblVdeaYk57itXrrQ6depYv379bP/997e77rrLli1bZnfeeadp3MRWq1Yt0zOKwLds2dKmTZvmUm/iW7I8+scee8wefvhhW7Fiheuqw4aeGzlypG9sRNx9I+MBCEAAAhAIkADCPUCYDBUjgHDPkc0wf/58O/zww6137972wAMP5Ft1u3btbMqUKbZ69WrbbbfdChTuBx98sH333Xd22WWXWY0aNZyI1zMS+9WqVXPCXm2nnXayjh07+hLuugjbo0cP69Chg7Vp08bKlCljX375pSnNZ8mSJb69hHD3jYwHIAABCEAgQAII9wBhMhTCPRf3wKGHHmpr1qxxf8qVK+cQrF+/3vbYYw9r27atu8CqVlDEXc8o1UZpMfGtoFQZPxH30047zZYvX26LFi3y7Zq1a9ea/sQ3if0uXbpY9W7DrHz1/Pb6noAHIAABCEAAAj4JINx9AqN7SgSIuKeEKRqdVP3l2muvtZdfftldVlVTesxVV11lEydONInnwoS7ouGeuA9auCvaLhveeOMNa9asmS/gAwcOtEGDBiV9BuHuCyWdIQABCEAgIAII94BAMkw+Agj3HNoQ33//vbtIKgH+wgsvuJU3atTIlYlU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e6x7adALxCGHHGJvvPGGrV271pYtW5ZH3JcuXWpt27a1Cy64wI488kj7559/TLXsb731lj311FN2xRVXuD5nz57tZP+dd96xZ599NhKP7lVdf7169axGjRpWt25d22effUwxjhkzxho0aFCg8BeXNsS9OEL8HgIQSBUBxD1VZOkXAhBA3DM4BwranOqFo9VmrUBLyqPFXavZWmUvU6ZMJPLt27fbtm3brHTp0nlGoxX6yZMn2y+//OLKaTZv3uxW77VC3q9fv0JH/t///teGDh3qVvdbtWoVue6SSy6xCRMm5BF3ifruu+/uXha8phr2Zs2a2ffff29ffvll5OdFlcroUwIJfHTTKnznzp2d7J9yyimFxqtPDvQnui1fvty9DFXpNMJKV6mRwSzzaAhAIGwEEPewZZzxQiB9BBD39LHe5UmeuPfu3dtatmzpfv/XX3+51eYHH3zQGjZsaNOmTXPS7a24a1X8hhtuKDRqCfzvv/9uknmtkl922WX20UcfuRV0lbHolJqmTZu60pTCaue1aq/2xRdf5HnOkiVL7KSTTsoj7tEXqH99YrBjxw5T3f6tt97qXha8UpdYatx1r+L/999/3d8q+fnf//5n1113XaFj1kr+oEGDCvw94p7BCc6jIRBSAoh7SBPPsCGQBgKIexogF/aIomrcdapMhw4d7PHHH7err746Iu4S+datW+/S5fjx423YsGGuRlzSHt1UW3766ae7H6luXi8KukZlKVoZ13OOO+64yC1azT/rrLPcS0N081bso2vctbousR45cqStWLFilxNjvv32W3e8pVpR4r548WLr37+/K7HZsmVLnudKym+//fZCM8WKewYnMY+GAAR2IYC4MykgAIFUEUDcU0U2hn6LEnfVnqvWW3XrL774YkTcVSsu2Y5ukyZNcue+a9VeNe/a2KlVeiX3pptusnnz5kWOm9R9KmF59dVXXe24NpXqWYMHD3bXqhUm7jqqskKFCnlW3B944AFXdqOae21g3X///V3pzGuvveZq5r3Np0WJu+ReJUDapKqXFP1dtmxZ93Jx9tlnF7rCXxRiatxjmIBcAgEIpIQA4p4SrHQKAQiYObfTvkCVTWsBNtlWYmc6DulONsqA3F+UuK9fv96VskhcdVqMnExLsAAAIABJREFUVypTkLhrg6fKYXSizG677RYZ3ahRo5wI5xf36OH//fff7rx4rXirNEV18vGUypxwwglO5jWW6HbLLbfYfffdl0fcVbajTwbyTxGt2Kv8J1ry1Zfq4xVL9Ap/rKlD3GMlxXUQgIDfBBB3v4nSHwQg4BFA3DM4F4oSd9WI6zhHCbCOeixK3LXKrkSuXLkysklUQl6/fn37/PPPI+Ku+nk1b8OrN3SdIa8TYPSyoNNl+vbt68puYtmcqmdoU2n0UY/q55hjjnH9Rcu4XiL0MqEjJPVpgtdUvqPTa3Sc5KGHHhr5uTbXatyIewYnKY+GAATiJoC4x42MGyAAgRgJIO4xgkrFZQV9c6q3OfXpp592Eq0EqfSlKHHXKrZWs1WqotV3ibGu33vvvd393oq7jn3Ueeo6ulFirU2j+qjliSeecKvu2syq9vPPP7uad9W0X3vtta50Rav+Ojde/WkzqGRa7e6773ZHTl588cXWpEkTd43kXOe969pocX/yySete/fuptNpVNajkhrV66tGvU6dOu45V111lfvUQKU8Og0n//GTseaBFfdYSXEdBCDgNwHE3W+i9AcBCHgEEPcMzoWCjoPUsYqSXm0OlSD/5z//cREWJe76/cMPP+z+rFmzxt2v1WqdSiMh98RdIqwz3fWlSrpOJ9Bo46g2p2qVPfo4Rq3e60uUdK/q5SXad911lztrPfo4SfWhn+uUmh9//NGtmPfs2dO9NCiGaHHXSTE6HUbnvGs1XiUz3u/nzJnjTqH59NNP3ScC55xzjql+XuVCrLhncJLyaAhAIG4CiHvcyLgBAhCIkQDiHiMoLjO3Oq/SGJ0vrxX2IDdW3IOcHWKDQG4TQNxzO7+MDgKZJIC4Z5J+gJ+tGnmd7OI1rY5rZX7KlClutb5atWoBjv7/7brmHPdAp4ngIJCTBBD3nEwrg4JAIAgg7oFIQ/CCOPnkk13duU6NkcSr5nzhwoXu9Bcd8xj0xop70DNEfBDIXQKIe+7mlpFBINMEEPdMZyCgz1fd+sSJE93qumrTDz/8cOvataurey9RokRAo/5/YSHugU8RAUIgZwkg7jmbWgYGgYwTQNwzngICSAUBxD0VVOkTAhCIhQDiHgslroEABBIhgLgnQo17Ak8AcQ98iggQAjlLAHHP2dQyMAhknADinvEUEEAqCPg9sVMRI31CAAIQgAAEIACBeAj47Tcldub/Pvt4ouFaCPhEwO+J7VNYdAMBCEAAAhCAAAQSJuC33yDuCaeCG/0k4PfE9jM2+oIABCAAAQhAAAKJEPDbbxD3RLLAPb4T8Hti+x4gHUIAAhCAAAQgAIE4CfjtN4h7nAng8tQQ8HtipyZKeoUABCAAAQhAAAKxE/DbbxD32NlzZQoJ+D2xUxgqXUMAAhCAAAQgAIGYCPjtN4h7TNi5KNUE/J7YqY6X/iEAAQhAAAIQgEBxBPz2G8S9OOL8Pi0EOMc9LZh5CARCTYDz2kOdfgYPgYwQQNwzgp2HppoA4p5qwvQPAQgg7swBCEAg3QQQ93QT53lpIYC4pwUzD4FAqAkg7qFOP4OHQEYIIO4Zwc5DU00AcU81YfqHAAQQd+YABCCQbgKIe7qJ87y0EEDc04KZh0Ag1AQQ91Cnn8FDICMEEPeMYOehqSaAuKeaMP1DAAKIO3MAAhBINwHEPd3Es/h5jRs3dtHPnz8/8KNA3AOfIgKEQNYTQNyzPoUMAAJZRwBxz7qU5Q341VdftXPPPdf+97//2XXXXZfnl4MGDbKBAwda+/bt7cUXX8zzuzlz5lizZs1s8ODBdvPNN8dEAXGPCRMXQQACISGAuIck0QwTAgEigLgHKBmJhLJp0yarVKmStW3b1iZPnpynizPPPNMWLlxo++23n61bty7P7+644w6788477e2337aGDRvG9GjEPSZMXAQBCISEAOIekkQzTAgEiADiHqBkJBrK8ccf78T8p59+inTx77//WsWKFd1q+7hx4+zLL7+0I488MvL7Jk2a2OLFi03iX6pUqZgejbjHhImLIACBkBBA3EOSaIYJgQARQNwDlIxEQ1GJzMMPP2zLly+3mjVrum7effddt5K+bNky9/dDDz1k3bp1c7/bunWrk/pTTjnFnn76aRs6dKjNnTvXvv32W/f7E044wW699VZr2bJlnpAKE/fp06fbAw88YJpM27dvN71I9O/f384555zI/atXr7ZDDz3U/fyYY46xu+++27755hv3MjFixAjTi8Sbb77pnvvJJ59YtWrV7L777nMvHok0atwTocY9EIBAPAQQ93hocS0EIOAHAcTdD4oZ7mPKlCnWrl07GzVqlF155ZUumiFDhjiZXr9+vZPigw8+2J555hn3O5XHNGrUyFQDX6tWLRswYIC1adPGDjvsMPvtt9/s2WeftU8//dRmz55tKrfxWkHi/sgjj1ivXr2sefPm1qpVKytRooRNmDDBreY///zz1qFDB3e7J+56KdiwYYP16NHD9thjDxfj77//7j4VuPbaa93P99lnH1ezrxeJFStWuLjibYh7vMS4HgIQiJcA4h4vMa6HAASSJYC4J0swAPdLzitXrmyXXnqpjR8/3kWk1W6VwLz88st22223ORmXPKtpQ6pWtufNm2cnnXSS7bnnnnlG8c8//7hVc8n+zJkzCxX3tWvXOqnu3r27SeC9plV3rfLr92vWrLHddtstIu577bWXK9s58MAD3eXe5trdd9/dfTpQu3Zt9/MPPvjA6tev7+K85557iqSsMqH8Nfz69KFjx45WpdMIK12lRgCyRAgQgECuEUDccy2jjAcCwSeAuAc/RzFFqJXzP/74w4nyjh07bN9993XC3qdPH5s1a5a1aNHCrWBLxlUCo9KYzZs3W5kyZSL9b9myxf7880/buXOnu/eFF16wX3/9tVBxV/nN9ddfb0uWLLHq1avnifOxxx4zbYDVyr1KY7wV94svvtitxHtNq+/777+/nX766bZgwYI8fZQtW9admKM4imo6OUefHhTUEPeYpg8XQQACCRBA3BOAxi0QgEBSBBD3pPAF5+arr77alcqsWrXKNm7caHXr1rX333/frairFEXlJ2PHjjWJs6T+2GOPtUWLFpk2sd51112ujMarcfdGpbIXvQR4LX+pzDXXXGOPP/54kRD0gqBSHU/cdfSkVvy9tm3bNvfJgFbH9alAdKtSpYrVqVPHlewU1VhxD848JBIIhIkA4h6mbDNWCASDAOIejDwkHYXqyi+55BJXKy5xV4mJVtRVgqJWr1499+eqq65yJSjaJKoNoj179nTyrdpy1b1L6kuWLGljxoxxK+NafS9M3L2XhalTp1q5cuUKHINeINRn9OZUPTe/uHfq1Mm9WOQXd5XOaNNqvI0a93iJcT0EIBAvAcQ9XmJcDwEIJEsAcU+WYEDu/+GHH1zdeNeuXZ24S9qjhfeGG26wN954w4l77969Xe36WWed5VbitTE1vzRrZX7ixIlFivuwYcOsb9++7gSbBg0aFEkCcQ/IRCEMCEDANwKIu28o6QgCEIiRAOIeI6hsuKxGjRpuI6ikXWUsqjH3mr6cSUcratOoTnyR3O+9997uy5m0kVUr9V7T5lFtTlXNe1Er7iqtOeKII0wlNDNmzNjlPPiff/7ZbZpVQ9yzYQYRIwQgEA8BxD0eWlwLAQj4QQBx94NiQPq44oorXImL2pw5c6xp06aRyPTlTKoZV1Pdu+rf1XS2u+7RSryOatTZ6iqd0WkxOuWlKHHX/Y8++qg7DlLnx+voR52/rtX/9957zz777LNI3TziHpBJQhgQgIBvBBB331DSEQQgECMBxD1GUNlwmVbNO3fu7Orateqe/5hHfdnRypUrXXmLzk9X00k0t9xyi6lOXSfISMC1gVTHKeqkluLE3XtJ0Jc46WVAp9IccMABbsVe5Tb6o4a4Z8MMIkYIQCAeAoh7PLS4FgIQ8IMA4u4HRfoIHAE2pwYuJQQEgZwjgLjnXEoZEAQCTwBxD3yKCDARAoh7ItS4BwIQiIcA4h4PLa6FAAT8IIC4+0GRPgJHAHEPXEoICAI5RwBxz7mUMiAIBJ4A4h74FBFgIgQQ90SocQ8EIBAPAcQ9HlpcCwEI+EEAcfeDIn0EjgDiHriUEBAEco4A4p5zKWVAEAg8AcQ98CkiwEQIIO6JUOMeCEAgHgKIezy0uBYCEPCDAOLuB0X6CBwBxD1wKSEgCOQcAcQ951LKgCAQeAKIe+BTRICJEPB7YicSA/dAAAIQgAAEIAABPwn47TcldkZ/Y4+fkdIXBOIg4PfEjuPRXAoBCEAAAhCAAARSQsBvv0HcU5ImOo2XgN8TO97ncz0EIAABCEAAAhDwm4DffoO4+50h+kuIgN8TO6EguAkCEIAABCAAAQj4SMBvv0HcfUwOXSVOwO+JnXgk3AkBCEAAAhCAAAT8IeC33yDu/uSFXpIk4PfETjIcbocABCAAAQhAAAJJE/DbbxD3pFNCB34Q4DhIPyjSBwTCQYBjHcORZ0YJgVwggLjnQhYZwy4EEHcmBQQgECsBxD1WUlwHAQhkmgDinukM8PyUEEDcU4KVTiGQkwQQ95xMK4OCQE4SQNxzMq0MCnFnDkAAArESQNxjJcV1EIBApgkg7pnOAM9PCQHEPSVY6RQCOUkAcc/JtDIoCOQkAcQ9J9PKoBB35gAEIBArAcQ9VlJcBwEIZJoA4l5MBsaOHWtdunSxVatWWfXq1TOdr7Q/f/78+dakSRMbPXq0devWLe3PT/SBiHui5LgPAuEjgLiHL+eMGALZSiB04u6JaHTC9tprLzv44IPtwgsvtD59+li5cuUiv860uK9evdoOPfTQPPNL8R177LF27bXXWocOHVI691Ih7g8++KDtu+++1rlz55TFjrinDC0dQyDnCCDuOZdSBgSBnCUQWnHv2rWrNW7c2CX2t99+s4ULF9oLL7xgzZs3t1mzZgVO3Nu0aWPt2rWznTt32tq1a+3JJ5+0r7/+2p5++mn3iUCqWirE/aCDDrIaNWqY+k5VQ9xTRZZ+IZB7BBD33MspI4JArhIIrbgXVPohMZ4yZYr99NNPVrlyZZfzoKy49+/f3+6+++7IPFSMhx9+uB1yyCH22WefpWx+Iu4pQ0vHEIBAQAgg7gFJBGFAAALFEkDcoxD16tXLHnnkEdu4caNVrFixUHH3VurzrxgXJvlr1qyxQYMG2euvv24bNmywAw880C655BK7/fbbrXTp0kUmySuVyS/uuunEE0+0jz/+2P75559IH5988omNGDHCfYKglfk99tjDTj75ZLvzzjvd3/nbW2+9Zffee6+999579vfff7uSoVatWtnw4cPdpYWJ+1133eXiv+2221zfanqB0DjnzZvnPsVQiU/37t3txhtvtN12281dU6JEiV1i0MuHxqn21FNP2aOPPmorV650/y5W4j1y5MhiJ3P0Bay4x4WLiyEQagKIe6jTz+AhkFUEQivuEtOOHTu6ZP3xxx/29ttvW48ePezMM8+0l156KZLEgmQ8HnH/5ptv7JRTTrFSpUq5zZ3VqlWzJUuW2JgxY6xFixY2ffr0AmXWC6Awcd+2bZv95z//cff+8MMPkXiHDh1qEydOtLPPPtutxmtlXjL8448/2gcffGC1atWKXKvSoEsvvdSqVq3q6s3Vn8pvXnzxRbcZtyBxV6nODTfcYA8//LB7QbjuuuvcdRL/Zs2amcpgOnXqZPvss48TePUlro899pi7bvz48e6eAw44wPQyorb33nvbeeedF/l0Q2VBYqOxid+rr75qy5cvj+s/LMQ9LlxcDIFQE0DcQ51+Bg+BrCIQWnEvKEsSxueff9723HNP38T9nHPOsU8//dSWLVvmNmR6TSv7WuF/4403nKQW1jxx7927t91yyy2RGvf777/fJkyYYPr5sGHDIrf/+eefps220U2r/BL2tm3b2qhRo9yvfv/9d7e6rpiWLl3qRNtrO3bsiKyQR6+4S+6vuOIK91zV1l922WXuFsm8Nsvq0wO9AEV/itC3b1/TZtTPP//catas6a4vrMZd8a1YsSLu0p9169aZ/kQ3ib5ezKp0GmGlq9TIqv8oCRYCEEgvAcQ9vbx5GgQgkDiB0Iq7hLdly5aO3F9//WUCIcFs2LChTZs2zZWYqCWz4r5p0yarVKmSW6GWdEe3X3/91Y466iiT2D7wwAPFinv+C0qWLOlOlZHAe7Hmv0bjUvmLxFrSrZV5jVNt6tSpdsEFFxR7zKMn7lph16bd2bNnu1X01q1bRx6ncp3jjjvOrcBrBT+6ffTRR24lXi8qPXv2LFLctclWcb322mt26qmnxjyrBw4c6Ep0CmqIe8wYuRACoSWAuIc29QwcAllHILTiXtDmVJWO6HjFxx9/3K6++uqkxX3x4sUF1pVHz5LLL7/cxo0bV6y4S4gl3//++6+Tb9Wln3/++fbMM8/kKbVRbfmAAQNs0qRJrjwmuqnmXKUnakOGDLGbb77ZFKNq5QtrnrjrCEqt0qu0R58iRDeJ/EUXXVTk5Fc9vCfXha24a7Vdnz7oUwZdo/Pj9YKgcepFpbDGinvW/e8OAUMgUAQQ90Clg2AgAIEiCCDuUXC0Qq6Skfbt27tVZbWCVtwllFrFzr85VbXkqmP3vqxJdd+qb9dLgFa3C2qqLz/mmGOKFff8m1NVrqINrloJ18q71yTVc+bMcRtC69ataxUqVHBlL4MHD3b1694m0HjFXc9S7b/6VHmP6tK9ppr6iy++2O644w5r1KhRgWPRS4NOwVEr6jjILVu2uJV9jUF/tOG1fv36tmDBgjwlTMX9V02Ne3GE+D0EIOARQNyZCxCAQLYQQNyjMrV+/Xp3DKQ2duoEmMLEXSvAkmCVgUQ3ybVWwj1xV3/ahHnllVfGfSpK5P9Q/v8vYCroVJkGDRrYV1995Z6nFXG9eKhmXavbKh+JbnqB0Mq0J+7xlsroEwptdj333HPtpJNOcuUsXi29Nr1KrvVyoFX84po2wUriYznHXS8m2swa73n1iHtxWeD3EIAA4s4cgAAEso0A4h6VMcmpJFv16BLwwsT9pptucvXckmCtmKtt3rzZjj76aCfHnrjr5yr9WLRokTvRRb+Pblpd3rp1q5UvX77QeVPUcZCvvPKKO43FE2aVsmiFXUc0Rtd8S5CbNm3qNqN64q6TdPTvqsHXKTfe8ZcKRJ8meMc25j8OcubMmaZNvNoLMGPGDCtbtqxpM6s+NdAxmnqZ0ctKdFNcOlWnTJky7sfapKoNrPlffLSJdr/99stzr46rPO2009w+AO0HiLUh7rGS4joIQIAVd+YABCCQLQRCK+7R35zqbU7Vqq7EUVCqVKlSqLhrlVvyqc2lV111ldvcKun3JDha3FVTro2Wqj3XiSy1a9c2nfzy5Zdf2uTJk923tWrzZmGtKHGXYEuYJby6TqfhqC+d7KITa4444gjTue46erJ69equRt0Tdz1Pz1YJjM5K18ZQlbDo9yp90ScKagWd467Vdp0AI6FWzbuE/P3333ffOitBF1s9WyKvE3W0uv/hhx+6b0tV02kvOr1HpTVHHnmkK7tRLbvKcMRQ5TaKRUdZ6hQcjU8bYL1Sm1j+40LcY6HENRCAgAgg7swDCEAgWwiEVtyjE6SNj1o5P+uss1yJiUo5vFbYlypJRlW+IsHVyrVqylU6IgGOFnf1o1V4reBLcvWlSFphV8236tFVn55/lTk6tqLEXdd58elEHMXw888/W58+fUwr43pB0Gkv+rKkZ5991kl4tLjrftWO33PPPU68tfFVY1Fc3hGThX0Bk8aiun2daa+Vf8m7vjRJfenkGZUJqWxHAq/yGo1Tq/Nq3333nXvh0Wq6Xia8L2DSy49eGiT7KvvZf//93UuPNtvWqVMnrv+mEPe4cHExBEJNAHEPdfoZPASyikDoxD2rskOwCRNA3BNGx40QCB0BxD10KWfAEMhaAoh71qaOwIsigLgzPyAAgVgJIO6xkuI6CEAg0wQQ90xngOenhADinhKsdAqBnCSAuOdkWhkUBHKSAOKek2llUIg7cwACEIiVAOIeKymugwAEMk0Acc90Bnh+Sggg7inBSqcQyEkCiHtOppVBQSAnCSDuOZlWBoW4MwcgAIFYCSDusZLiOghAINMEEPdMZ4Dnp4QA4p4SrHQKgZwkgLjnZFoZFARykgDinpNpZVB+T2yIQgACEIAABCAAgUwT8NtvSuzU13nSIJBhAn5P7AwPh8dDAAIQgAAEIAAB89tvEHcmVSAI+D2xAzEogoAABCAAAQhAINQE/PYbxD3U0yk4g/d7YgdnZEQCAQhAAAIQgEBYCfjtN4h7WGdSwMbt98QO2PAIBwIQgAAEIACBEBLw228Q9xBOoiAO2e+JHcQxEhMEIAABCEAAAuEi4LffIO7hmj+BHS3HQQY2NQQGgYwR4NjHjKHnwRCAgE8EEHefQNJNsAgg7sHKB9FAIAgEEPcgZIEYIACBZAgg7snQ497AEkDcA5saAoNAxggg7hlDz4MhAAGfCCDuPoGkm2ARQNyDlQ+igUAQCCDuQcgCMUAAAskQQNyToce9gSWAuAc2NQQGgYwRQNwzhp4HQwACPhFA3H0CSTfBIoC4BysfRAOBIBBA3IOQBWKAAASSIYC4J0MvoPfOnz/fmjRpYqNHj7Zu3bqlNcrq1atb48aNbezYsWl9bv6HIe4Zxc/DIRBIAoh7INNCUBCAQBwEEPc4YMV6qSfOun7IkCHWr1+/XW4dMWKE3Xjjje7ns2fPtmbNmsXafbHXIe5miHux04QLIBA6Aoh76FLOgCGQcwQQ9xSk1BPnMmXKWI0aNeyTTz7Z5Sl169a15cuX25YtWxD3FOQAcU8BVLqEQJYTQNyzPIGEDwEIRBYmP/jgA5NLJtv4AiYz88S9ffv2NmnSJFu2bJkdf/zxEbafffaZ1a5d2y688EJ78cUXfRP3v/76y/SysHDhQkpl/u//tXr16lmVTiOsdJUayc5r7ocABHKAAOKeA0lkCBAIOQFW3FMwATxxHzVqlA0aNMg6dOhgw4YNizxJpTPPPPOM3X333da9e/c84i75VnnNxIkTbfXq1VahQgU7++yz7d5777WDDjoo0sfAgQNd30qgatmnTJli69evt19//dU+/PDDAsX9rrvusttvv91uu+02u/POO11fTz31lD366KO2cuVK9+8HHnigq1EfOXJk5FklSpSwSy+91P3p37+/ff7551atWjW77rrr7IYbbshDMH+N+9atW13sr7/+un311Vf2559/2mGHHWZXXnmlXX/99aa+vbZhwwbTuKZPn27r1q2z8uXL21FHHeWu00tQPI0V93hocS0EwkEAcQ9HnhklBHKZAOKeguxG15h/+eWXNn78ePv++++tZMmStmPHDjv44IOdiB533HHWpUuXiLhLcrWpVCv0Xbt2tTp16tiaNWucWJcrV85J+n777eci9sRdK/f62fnnn2+///67E+nFixfnEfedO3e6nz/88MOm2noJt5o2kOr5bdq0sRYtWjiJ/uabb+zVV191ZTxe08+POeYY++677+zqq692cq9PEt566y2777777Kabbopcm1/cJeM1a9Z049XfYjBz5kwn59EvEOpALwwa4zXXXGNHHHGEbdq0yb2EaHzDhw+P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V0MAArlBAHHPjTwyinwEEHemBARymwDintv5ZXQQgEDBBBB3ZkZOEkDcczKtDAoCEQKIO5MBAhAIIwHEPYxZD8GYEfcQJJkhhpoA4h7q9DN4CISWAOIe2tTn9sAR99zOL6ODAOLOHIAABMJIAHEPY9ZDMGbEPQRJZoihJoC4hzr9DB4CoSWAuIc29dkx8BIlStgdd9xhAwcOjCtgxD0uXFwMgawjgLhnXcoIGAIQ8IEA4u4DxKB0MX/+fGvSpEkknN13390qVKhghx12mJ166qnWtWtXq127dlDCjSkOxD0mTFwEgdARQNxDl3IGDAEImBninkPTwBN3CXrjxo1tx44dtnnzZvv4449typQp7p9vu+22uFevM4kIcc8kfZ4NgeASQNyDmxsigwAEUkcAcU8d27T37In76NGjrVu3bnmeL2m/6KKLbObMmfb0009bly5d0h5fIg9E3BOhxj0QyH0CiHvu55gRQgACuxJA3HNoVhQl7hqm5L169equfGbVqlUmKVabPn26PfDAA+7jl+3bt9vxxx9v/fv3t3POOScPnaeeesoeffRRW7lypfv5gQce6Fb2R44cGbnu5Zdftvvvv9+WL19uW7dutSpVqrgyHd1Xrly5yHXvvPOO3X333fbuu+/a33//bTVr1rQbb7zROnXqlOeZiHsOTVCGAgEfCSDuPsKkKwhAIGsIIO5Zk6riAy1O3NXDFVdcYWPGjLHPP//cjj76aHvkkUesV69e1rx5c2vVqpWT+QkTJtjixYvt+eeftw4dOrgHjx071q3St2nTxlq0aOGu++abb+zVV191kq42d+5ca9asmZ1++ul2wQUXWJkyZezbb791LwYzZsxwoq8muW/fvr3VrVvX2rVrZ3vuuadNmzbNZs2aZUOGDLF+/fpFBou4F593roBAGAkg7mHMOmOGAAQQ9xyaA7GI+/Dhw6137972yiuvWL169dzG1e7duzuB95pW3Rs2bGhr1661NWvW2G677WZt27a1FStW2GeffVYoMa2Yqwznl19+MW2MLahpdf3ggw+2Bg0aOFn3Vv11rST+tddesx9++MEqVqzobo9F3NetW2f6E930MtGxY0er0mmEla5SI4eyzFAgAAERQNyZBxCAQBgJIO45lPVYxP3JJ590oj5+/Hgn2Ndff70tWbLEldBEt8cee8wdw/jpp5/aMccc41bb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DyCAAAIIIIBAJQUI3CvZrFQqdsdGFAEEEEAAAQQQSC0QO75hxj11i7J/ZtzpAwgggAACCCBQSQEC90o2K5UaNWqU9e7d24YPH27du3cHBAEEEEAAAQQQKL3A6NGjbcCAATZy5Ejr1atX0/Vhxr1pQjYQQ2DIkCE2cODAGJtiGwgggAACCCCAQKEENDHZv3//po+JwL1pQjYQQ2DEiBHWp08fGzZsmPXo0ZRRb5wAAA6GSURBVCPGJku3jWxU3upPHXAwwwADXcDoBxjQD/69lZf1fBg3bpyNGTPG+vbta126dGk6NiFwb5qQDcQQiJ0DFuOYvLeBwT/iOGBAP+BcyK6/XA+4HtAX2kcjBO7e0Rn7ayjAxZmLMxfnf08NzgfOBwYvDF64JrYPF7gu/uNB4E4gXQgBTkgCFW5SBO61FyOuCVwTGLwweOGaMH6IRuBeiLCVg+AmzU2awJ3AnZs0M4z1d0PuDdwbuDeQKkOUXECBsWPH2tChQ2333Xe3rl27FvAI8z8kDP4xxgED+gHnQnbF5XrA9YC+QOCefwTGHhBAAAEEEEAAAQQQiCxAqkxkUDaHAAIIIIAAAggggEAeAgTueaiyTQQQQAABBBBAAAEEIgsQuEcGZXMIIIAAAggggAACCOQhQOCehyrbRAABBBBAAAEEEEAgsgCBe2RQNocAAggggAACCCCAQB4CBO55qLbgNv/44w877bTT7JJLLrEPP/zQ5plnHttll13s4IMPtskmm+w/RV577TUbNGiQjRw5Mvy2d+/eduqpp9riiy8+3t925rf/ueOIP/AyeOCBB+ymm26yF154wV599VX77bff7P3337f55psvYm0mblMeBj///LNdeeWVdscdd4T6f/XVV6HuG2ywgR1++OE288wzT9zBR/wrDwcdrs6RO++809566y377rvvbI455rAVV1zRjjjiCOvRo0fEGnV+U14G9Ue22mqr2eOPP279+/e34cOHd/7AI/6Fl8EOO+xgV1xxRcMj1/V47rnnjlirzm3KyyA7quuuu86GDBlir7zyiv3111+24IIL2p577hmWGk5VPAzGjBlj888/f4dVnHzyycO9ImXxcMjqd/XVV9u5554bro3qB926dbPddtstxCWTTjppSoam903g3jQhG5DAXnvtZRdccIHtuOOOtvLKK9uTTz5pl112Wfjv55133gSR3n77bVt++eVt1llntYEDB4bfnnPOOfbtt9/as88+awsttFDb33fmt94t42Wgm/S1115rSyyxhP3yyy+mgUxRAncPA9V3ySWXtFVWWcX69u1rs88+exjEaNCoAF7/POOMM3o3f7v9eThoh5tvvrnNNNNM1r17d5tlllnso48+CufdJ598Yo8++qittNJKyRy8DGorqAGd9vvTTz8VInD3MsgCdwXv9UHJpptuatNOO21L9IMDDjjAzj77bNtiiy1MAzgFbLpnTD311HbSSSdV2kB9/tZbbx2vjhq4aUJjww03DJMdKYvX+XDiiSfakUceGe4PG220UegHN998sz3yyCN24IEH2umnn56Soel9E7g3TcgGNOu51FJLhaBbF82s7LfffmHE+/LLL4cgs6Oy2Wab2X333WejR48OM/UqutgoGFl33XXtxhtvbPvTzvzWs2U8DT7++GPr0qWLTTXVVOHipItUEQJ3L4Mvv/wyBKYK3mvLpZdeajvvvLOdccYZpht4quLl0FH9Pv3003Ae6VzRAC9FSWGggf4iiyxi+++/vx122GHJA3dPgyxw14yqZlaLUjwN7r777vDU7ZprrrGtt966KAThqaDX/bFRpU844QQ76qijQuCqQVyq4umgyZx55503TPxNMskkocp//vmnLbPMMqYnE7pWlLkQuJe59Qpy7Hosr9mM9957r92jOgWTCyywQBjtK7hsVH788UebbbbZbKutthrvUe/2229v119/vSlQm3766a0zv/Wm8TKor1eRAvdUBpnJ999/H2afd9ppJxs2bJh3F2jbX2oH3aCULtSrVy+79957kzikMNh7773twQcfDE+gNKhNnSrjaZAF7r/++quNGzcuXC+LkA7gabDqqqua0uief/75MMOq+8UMM8yQpP/X7tTToFFlF154Yfv666/DZMeUU06ZzMPTYZppprE111zT7rrrrnb11Qy8BhCyKHMhcC9z6xXk2HUyaFZdM331RTm3Sy+9dJhRb1SeeuqpkFqjNJs99tij3U/03/RoTb/p2bNn+P//+1tvGi+DIgfuqQwyE+UyLrroonbooYfa4MGDvbtA2/5SOGhwq4BdN6QzzzwzvANw1llnmZ56pSjeBkqPWmGFFcKNWk/pNMuWOnD3NMgCdwWqP/zwgylwkYPegVCOd6riZaAgXYN25bLraaRSLb/55puQ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" width="600">


De cijfers tonen dus dat 5 procentpunten een typisch thuisvoordeel is, en dat TTK Minderhout geen uitzonderlijk thuisvoordeel heeft. De clubs met aan de extremen zijn overigens relatief kleine clubs met weinig officiele wedstrijden. Toevoeging van enige onzekerheidsbanden zou interessant zijn, maar is voor een andere keer. Mogelijks dat Salamander als grote club met een groot thuisvoordeel dat bovenaan komt, maar die zijn enkele jaren van zaal gewisseld, dus ook dat zou dan uitgespit moeten worden.

## 3e provinciale en hoger
Om nog net een beetje dieper te graven, heb ik bekeken of we met TTK Minderhout dan wel een significant thuisvoordeel zouden hebben als we enkel de hogere provinciale reeksen in beschouwing nemen. Misschien dat een bepaald speelniveau nodig is om een thuisvoordeel maximaal uit te buiten? De details heb ik uit de post gelaten, maar de conclusie van een analyse beperkt tot 1e, 2e en 3e provinciale was hetzelfde: Geen significant thuisvoordeel. Omdat data waarover we statistiek doen kleiner wordt, zien we wel dat de extremen groter worden, met Dylan Berlaar die tot 10 procentpunten thuisvoordeel in die reeksen behaalt.

# Bonus: Vlaamse spelers met meeste individuele overwinningen in seniorencompetitie

In al de seizoenen dat ik bij Minderhout competitie heb gespeeld, heb ik er veel als kopman gefungeerd, waarbij ik in een lagere reeks speelde dan ik misschien zou aankunnen en hoge percentages (met regelmatige individuele leidersplaatsen in mijn reeks). Als dat geen clubliefde is!

Bovendien speel ik de meeste seizoenen bijna alle wedstrijden. Ik had dus het vermoeden dat er niet veel spelers in Vlaanderen meer individuele overwinningen in de enkelcompetitie zouden hebben dan ik. Met de TabT API werkend, de uitgelezen kans om dit eens te checken!

## GetMembers API

De GetMembers API ziet er als volgt uit:

	Array
	(
		[Credentials] => Credentials Object
			(
				[Account] => 
				[Password] => 
			)

		[Club] => A135
		[Season] => 21
		[PlayerCategory] => 
		[UniqueIndex] => 
		[NameSearch] => 
		[ExtendedInformation] => 0
		[RankingPointsInformation] => 
	)
	
Deze staat toe om individuele spelers op te vragen, inclusief al hun individuele resultaten.

Eerst vullen we een dictionary met alle spelers bij een club


```python
clubInds = []
for club in clubs.ClubEntries:
    #if 'A' in club.UniqueIndex:
    clubInds.append(club.UniqueIndex)

club_to_members = {}
for club in clubInds:
    try:
        club_to_members[club] = client.service.GetMembers(credentials, club, 20, None, None, None, 0, None)
    except:
        continue
```

Vervolgens vullen we een dictionary voor elke speler met al zijn individuele resultaten over alle beschikbare seizoenen. Spelers hebben een unieke lidnummer. Vanuit mijn jaren als kapitein zijn toen wedstrijdladen nog op papier ingevuld moesten worden, ken ik die nog vanbuiten: 502575


```python
member_to_results = {}

for club in club_to_members:
    #members = club_to_members[club]
    print(club)
    
    for season in range(7, 21):
        
        success = False
        tries = 0
        members = None
        while (not success) and (tries < 5):
            try:
                tries += 1
                members = client.service.GetMembers(credentials, club, season, None, None, None, 0, None, True)
                success = True
            except Exception as e:
                if not e.message.startswith('Quota'):
                    print(e.message)
                    members = None
                    success = True
                else: 
                    time.sleep(30.)
        
        if tries >= 5:
            print('Exceeded # of tries')
            
        if not members:
            continue
        
        assert out.MemberCount <= 1
        if out.MemberCount == 0:
            continue
            
        for member in members.MemberEntries:
            index = member.UniqueIndex
        
            if not index in member_to_results:
                member_to_results[index] = [0, 0] # Wins, losses
                
            results = member.ResultEntries
            for result in results:
                
                if not result.CompetitionType == 'C':
                    continue
                
                assert result.Result in 'VD'
                
                if result.Result == 'V':
                    member_to_results[index][0] += 1
                if result.Result == 'D':
                    member_to_results[index][1] += 1
```

## Resultaten

Deze keer presenteren we het wat mooier in een ```pandas``` tabel


```python
import pandas as pd

import copy

member_to_results2 = copy.deepcopy(member_to_results)

for club in club_to_members:        
    members = club_to_members[club].MemberEntries
    
    for member in members:
        index = member.UniqueIndex
        if not index in member_to_results2:
            continue
            
        if len(member_to_results2[index]) == 2:
            member_to_results2[index].append(member.FirstName)
            member_to_results2[index].append(member.LastName)

df = pd.DataFrame.from_dict(member_to_results2, orient='index', columns=['W', 'L', 'FirstName', 'LastName'])
```


```python
df.sort_values('W', ascending=False)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>W</th>
      <th>L</th>
      <th>FirstName</th>
      <th>LastName</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>500376</th>
      <td>1039</td>
      <td>731</td>
      <td>JACQUES</td>
      <td>INGELBRECHT</td>
    </tr>
    <tr>
      <th>502575</th>
      <td>947</td>
      <td>199</td>
      <td>LENSE</td>
      <td>SWAENEN</td>
    </tr>
    <tr>
      <th>506488</th>
      <td>944</td>
      <td>291</td>
      <td>MICHEL</td>
      <td>MENTENS</td>
    </tr>
    <tr>
      <th>503221</th>
      <td>914</td>
      <td>128</td>
      <td>ROB</td>
      <td>WUYTS</td>
    </tr>
    <tr>
      <th>506188</th>
      <td>879</td>
      <td>348</td>
      <td>JAN</td>
      <td>LEBBE</td>
    </tr>
    <tr>
      <th>505690</th>
      <td>856</td>
      <td>607</td>
      <td>ANDRE</td>
      <td>AUDEZ</td>
    </tr>
    <tr>
      <th>506952</th>
      <td>832</td>
      <td>352</td>
      <td>TOMMY</td>
      <td>VOET</td>
    </tr>
    <tr>
      <th>100577</th>
      <td>829</td>
      <td>334</td>
      <td>FRANCOIS</td>
      <td>GOBEAUX</td>
    </tr>
    <tr>
      <th>509041</th>
      <td>828</td>
      <td>358</td>
      <td>JORNE</td>
      <td>BICKX</td>
    </tr>
    <tr>
      <th>510800</th>
      <td>824</td>
      <td>204</td>
      <td>JOHAN</td>
      <td>CARNA</td>
    </tr>
  </tbody>
</table>
<p>72832 rows × 4 columns</p>
</div>



Aanvoerder van bovenstaande lijst is [Jacques Ingelbrecht](https://competitie.vttl.be/?season=22&sel=10480&result=1&category=1), 70-jarige speler bij het West-Vlaamse Zandvoorde. Hij voert bovenstaande lijst aan met 1039 overwinningen op meer dan 1700 wedstrijden. De reden dat Jacques zoveel wedstrijden op zijn teller heeft, is omdat hij als veteraan ook al jaren meedraait in de West-Vlaamse veteranen competitie, zodat hij wekelijks, naast 4 wedstrijden seniorencompetitie ook nog eens 3 wedstrijden veteranencompetitie speelt. Deze had ik er niet uitgefilterd...

Als we die wedstrijden negeren (en toegegeven, zonder veteranen-, toernooi- en jeugdwedstrijden wordt het een beetje cherrypicking :)), dan voer ik weldegelijk de Vlaamse lijst aan van spelers met de meeste individuele overwinningen in seniorencompetitie! Joepie!

Verder nog een shout-out naar collega Antwerpenaren Rob Wuyts en Jan Lebbe die mee de top aanvullen. Tegen Rob en Jan heb ik de voorbije jaren al vele wedstrijden op het scherpst van de snee gespeeld en beide hebben duidelijk een even groot hart voor hun kleine club getoond (Hulshout en Hove respectievelijk).

Tot slot de observatie dat mijn teller van overwinningen de 1000 aan het naderen is! Dat had ik eerder nog niet op mijn radar, maar dit geeft me dus nog ongeveer 1 volledig seizoen de tijd om een gepaste viering voor die mijlpaal te plannen!

# Verdere ideeen:

Nog enkele ideeen voor toekomstige analyses en blogposts die ik alvast even neerpen hier:
   - Toevoeging van onzekerheidsbanden aan de resultaten.
   - Uitnadeel ipv thuisvoordeel: Bij welke clubs presenteren bezoekers minder goed dan bij andere? En schept dit uberhaupt een ander beeld dan de vraag naar thuisvoordeel?
   - Wat voor verdelingen van matchscores (individueel dan wel op team niveau) vinden we terug? Wie zijn de belle-kampioenen en kneuzen?
   - Elo vs Glicko vs Trueskill rating systemen: Naast de officiele klassementen van de VTTL/KBTTB, kan je op de Frenoy-website ook Elo-ratings vinden. Het voorbije jaar heb ik me wat verdiept in verschillende ratingsystemen, zowel de praktische zoals die gehanteerd worden door de VTTL, Badminton Vlaanderen, Tennis Vlaanderen, Nederlandse tennisbond, etc. alsook de theoretisch gevestigde waarden als Elo, Glicko en Trueskill. Bij uitstek deze laatste is wiskundig erg interessant als toepassing van 'expectation propagation' type message passing voor inferentie op Bayesiaanse modellen. Dit heeft ook veel professionele relevantie voor mij, dus daar wil ik zeker nog een keer meer over schrijven. In elk geval, Trueskill heeft de belofte om sneller accuratere ratings te geven dan Elo, en onderbouwt dit onder meer met data uit online gaming. Interessant om een keer op al deze VTTL data te testen dan!

Als enige lezers nog suggesties hebben voor analyses, ben ik zeer geinteresseerd. Hopelijk dat de beschikbaarheid van de code in deze post ook andere in staat kan stellen om zelf wat interessants uit die grote hoeveelheid data op te graven.

-----

## Related posts
None yet

## Want to leave a comment?
Very interested in your comments but still figuring out the most suited approach to this. For now, feel free to send me an [email](mailto:lenseswaenen@gmail.com).