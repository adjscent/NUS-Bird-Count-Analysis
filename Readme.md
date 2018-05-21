#Bird Analysis Documentation
@authors Evan, Jason, Yixuan
##Data Read This is to feed the csv from the observation data pool to our data analysis engine.
code
# The usual preamble

import matplotlib.pyplot as plt
import numpy as np
import pandas as pd

# Make the graphs a bit prettier, and bigger
pd.set_option('display.height', 1000)
pd.set_option('display.max_rows', 500)
pd.set_option('display.max_columns', 500)
pd.set_option('display.width', 1000)


#read data
df = pd.read_csv("/Users/jason/Downloads/data.csv")
List Possible Misspellings
We noted that the observers really just hate this assignment
code
df.loc[(df['Bird Species'].str.upper().str.contains('JAVAN'))]['Bird Species'].value_counts()
output
Javan Myna              473
Javan Myna               81
Javan Mynah               2
Javan myna                2
Javan Myna (x2)           1
Javan Myna x4             1
javan Myna                1
Suspected Javan Myna      1
List Unknown Sightings
We noted several observers were unable to name some bird sightings
code
df.loc[(df['Bird Species'].str.upper().str.contains('\?|SUSPECTED|\`|^-$|^BIRD'))]['Bird Species'].value_counts()
output
Large grey/black bird, suspected pigeon    3
Bird that sounds like monkey               2
Bird of prey(?)                            2
Black-naped oriole (?)                     1
-                                          1
Olive-backed sunbird (?)                   1
Bird that makes a low-pitched quack        1
pigeon(?)                                  1
swiflet (?)                                1
`                                          1
Olive-backed Sunbird (?)                   1
Suspected Javan Myna                       1
List Possible Distances
We noted non-standard distance recordings code
df['Distance Bin'].value_counts()
output
2               550
1               469
Flyby           253
Heard           208
3                85
In Flight        77
flyby            56
4                42
Did not land     20
in flight        14
Flyby (4)         4
heard             4
flyby(3)          2
-                 2
In flight         1
Possible Habitat Types
We noted non-standard habitat type recordings
code
df['Habitat Type'].value_counts()
output
Urban                                1276
Tree                                   98
Forest                                 38
Urban (With tall trees around)         34
Field                                  32
urban                                  19
Urban (Tree)                           16
urban with some vegetation             15
Garden                                 14
garden                                 11
Field                                  10
Tree (10m high in tree)                 9
field                                   9
Urban with vegetation                   9
Garden/Park                             8
Urban (Small tree)                      7
Urban (Pavement)                        6
Urban (Rooftop)                         6
Shrubs                                  6
Pavement                                6
forest                                  5
NIL (flight)                            5
Road                                    4
NIL (in flight)                         4
Urban (Indoor carpark)                  3
Under tree                              3
Tree (20m high in tree)                 3
Urban/Forest Boundary                   3
Tree (5m high in tree)                  3
Trees at side of town green             2
Tree opposite road                      2
Tree along pavement                     2
Trees right outside Pizza Hut           2
NIL(in flight)                          2
Grass                                   2
Pavement alongside shrubs               1
Tree next to Supermarket                1
Covered walkway                         1
Tree (30m high in tree)                 1
Heard only                              1
Tree (15m high in tree)                 1
On town green                           1
Angsana tree canopy                     1
Urban (In Drain)                        1
Tree with loamy ground                  1
Poolside                                1
Urban (Under shade of building)         1
Urban (Building; 3rd Storey)            1
On town green closer to signboard       1
Tree (8m high in tree)                  1
Tree next to pavement                   1
Tree in front of Pizza Hut              1
Building opposite bus stop              1
Urban (Top of lamppost)                 1
NIL (heard only)                        1
Top of Angsana tree                     1
##Data Cleanup Data does not conform to the standards requested in the field guide. This section will normalise most of the data so that it can be used for further analysis
code
# Some guys did not fill in any data... sigh
df = df.fillna("")

# fix idiots' mistakes
target_coords = (
    '1.303894, 103.775190',
    '1.305994, 103.773311',
    '1.306042, 103.774040',
    '1.306989, 103.773073')

lagging_pointer = {'coords': '', 'loc': ''}  # tuple< current coord, current location >
for _, row in df.iterrows():
    current_pointer = {'coords': row[4], 'loc': row[5]}
    if current_pointer['coords'] in target_coords:
        if current_pointer['coords'] == lagging_pointer['coords']:
            row[5] = lagging_pointer['loc']
    lagging_pointer = {'coords': row[4], 'loc': row[5]}


#one dumb guy typo
for _, row in df.iterrows():
    if 'In the middle of YIH and CLB bus stop, opposite Engineering E' in row[5]:
        row[5] = 'In the middle of YIH and CLB bus stop, opposite Engineering E4'   
    

# cleanup whitespaces
df.replace('^\s+', '', regex=True, inplace=True)  #front
df.replace('\s+$', '', regex=True, inplace=True)  #end

# replicate rows based on x?
df['Count'] = df['Bird Species'].apply(str.upper).str.replace(' ', '').str.extract('X(\d*)', expand=True).fillna(
    1).astype(int)
df = df.loc[np.repeat(df.index.values, df['Count'])]
df['Count'] = None

# format bird name
df['NBirdSpecies'] = df['Bird Species'].apply(str.upper).str.replace('X(\d*)', '').str.replace('\(\)', '').apply(
    str.strip)

# format Habitat Types
df['Habitat Type'] = ["forest" if "tree" in x.lower() else x for x in df['Habitat Type']]

df['Habitat Type'] = ["urban" if "urban" in x.lower() else x for x in df['Habitat Type']]

df['Habitat Type'] = ["urban" if "pavement" in x.lower() else x for x in df['Habitat Type']]

#fix some guys' wrong spelling

df['NBirdSpecies'] = ["SWIFTLET" if "SWIF" in x else x for x in df['NBirdSpecies']]

for idx, row in df.iterrows():
    if 'JAVAN MYNAH' in row[11]:
        row[11] = 'JAVAN MYNA'
    elif 'SWIF' in row[11]:
        row[11] = 'SWIFTLET'
    elif 'OLVE-BACKED SUNBIRD' in row[11]:
        row[11] = 'OLIVE-BACKED SUNBIRD'
    elif 'PINK NECK GREEN PIGEON' in row[11]:
        row[11] = 'PINK-NECKED GREEN PIGEON'
    elif 'JARVAN MYNA' in row[11]:
        row[11] = 'JAVAN MYNA'
    elif 'OLIVE-WINGED BULBOL' in row[11]:
        row[11] = 'OLIVE-BACKED SUNBIRD'
    elif 'ROCKED PIGEON' in row[11]:
        row[11] = 'ROCK PIGEON'
    elif 'PINK NECK GREEN PEGION' in row[11]:
         row[11] = 'PINK-NECKED GREEN PIGEON'

        

# format time
def fix_time(y):
    y = y.replace(' ', ':')
    y = y.replace('.', ':')
    if len(y) == 1:
        y = "0" + y + "00"
    if len(y) == 2:
        y = y + "00"
    if len(y) == 3:
        y = '0' + y
    if ':' not in y:
        y = y[:2] + ':' + y[2:]
    if y[-3] != ':':
        y = y + "0"
    return y


df['Time'] = df['Time'].apply(fix_time)
df['Time'] = pd.to_datetime(df['Time'], errors='coerce')
Get number of unique locations
This is to check if the locations have reached 200. Please look at the next section for more details
code
df['Location'] = df['Location'].str.lower()
df.drop_duplicates('Location')['Group Name'].value_counts()

print('Number of unique locations: {}'.format(
    len(set(df['Location'])))) 
output
Number of unique locations: 194
Get number of group locations
This is to check if every observer group has recorded down their observations. As noted, several groups did not record down all locations as some locations do not have any bird sightings
Code

asian_koel = df[(df['Group Name'] == 'Asian Koel')]
ashy_minivet = df[(df['Group Name'] == 'Ashy Minivet')]
banded_woodpecker = df[(df['Group Name'] == 'Banded Woodpecker')]
brahminy_kite = df[(df['Group Name'] == 'Brahminy Kite')]
collared_kingfisher = df[(df['Group Name'] == 'Collared Kingfisher')]
coppersmith_barbet = df[(df['Group Name'] == 'Coppersmith Barbet')]
crimson_sunbird = df[(df['Group Name'] == 'Crimson Sunbird')]
pacific_swallow = df[(df['Group Name'] == 'Pacific Swallow')]
spotted_dove = df[(df['Group Name'] == 'Spotted Dove')]
spotted_wood_owl = df[(df['Group Name'] == 'Spotted Wood Owl')]

groups = (asian_koel, ashy_minivet, banded_woodpecker,
          brahminy_kite, collared_kingfisher, coppersmith_barbet,
          crimson_sunbird, pacific_swallow, spotted_dove, spotted_wood_owl)

print('Number of rows: {}'.format(len(df)))
for group_num, group in enumerate(groups):
    print('Number of rows by group {}: {}'.format(group_num+1, len(group)))
    print('                locations: {}'.format(len(set(group['Location']))))
output

Number of rows: 1788
Number of rows by group 1: 125
                locations: 20
Number of rows by group 2: 99
                locations: 20
Number of rows by group 3: 173
                locations: 19
Number of rows by group 4: 209
                locations: 20
Number of rows by group 5: 132
                locations: 18
Number of rows by group 6: 381
                locations: 20
Number of rows by group 7: 170
                locations: 20
Number of rows by group 8: 217
                locations: 20
Number of rows by group 9: 150
                locations: 20
Number of rows by group 10: 132
                locations: 19
                
##Drop all other birds except the 5 species
Observers have recorded down other birds. Due to the premise of this study, we will only take the 5 specified bird species.
df = df.loc[(df['NBirdSpecies'] == 'JAVAN MYNA') |
              (df['NBirdSpecies'] == 'YELLOW-VENTED BULBUL') |
              (df['NBirdSpecies'] == 'OLIVE-BACKED SUNBIRD') |
              (df['NBirdSpecies'] == 'BLACK-NAPED ORIOLE') |
              (df['NBirdSpecies'] == 'ROCK PIGEON')]
##Drop all flyby
Observers have recorded down flybys, which is not needed for in population estimates.
df = df.loc[(df['Distance Bin'].str.lower().str.contains("^1$|^2$|^3$|^4$"))]
##Count by Habitat
Breakdown of bird observations by the 4 specified habitats. We have noted that several observers have creative writing so we have normalised their habitat recordings.
df[df['Habitat Type'].fillna("").str.lower().str.contains('forest')]['NBirdSpecies'].value_counts()

YELLOW-VENTED BULBUL    53
JAVAN MYNA              48
OLIVE-BACKED SUNBIRD    41
BLACK-NAPED ORIOLE      17
ROCK PIGEON              6

df[df['Habitat Type'].fillna("").str.lower().str.contains('urban|pavement|building|shrubs|walkway|road')]['NBirdSpecies'].value_counts()

JAVAN MYNA              372
YELLOW-VENTED BULBUL    226
OLIVE-BACKED SUNBIRD    177
ROCK PIGEON              87
BLACK-NAPED ORIOLE       79

df[df['Habitat Type'].fillna("").str.lower().str.contains('garden|field|green|grass')]['NBirdSpecies'].value_counts()

JAVAN MYNA              31
YELLOW-VENTED BULBUL    30
OLIVE-BACKED SUNBIRD     5


df[~(df['Habitat Type'].fillna("").str.lower().str.contains('garden|field|green|urban|pavement|garden|field|green|tree|forest|walkway|road|shrubs|building|grass'))]['NBirdSpecies'].value_counts()

JAVAN MYNA              10
YELLOW-VENTED BULBUL     4
OLIVE-BACKED SUNBIRD     5
BLACK-NAPED ORIOLE       0
Overall Population Count
Code
ptCount = df['NBirdSpecies']\
           .value_counts() 

print(ptCount)
print(df['NBirdSpecies'].count())
Output
JAVAN MYNA              461
YELLOW-VENTED BULBUL    313
OLIVE-BACKED SUNBIRD    228
BLACK-NAPED ORIOLE       96
ROCK PIGEON              93
1191
Overall Population Density
Code
numOfPt = 200
popCount = ptCount / (0.01 * numOfPt * np.pi)
print(popCount)
print(1191 / (0.01 * numOfPt * np.pi))
Output
JAVAN MYNA              73.370429
YELLOW-VENTED BULBUL    49.815497
OLIVE-BACKED SUNBIRD    36.287327
BLACK-NAPED ORIOLE      15.278875
ROCK PIGEON             14.801410
189.55353722244735
Overall Population Density Graph
Code
pol = popCount.plot(kind='barh')
pol.grid()
plt.tight_layout()
plt.show()
Locations observed over Time Graph
pol = df.drop_duplicates(['Location']).set_index('Time')[['Location']].groupby(pd.TimeGrouper(freq='10Min')).count().plot(kind=’barh’)
pol.grid()
plt.tight_layout()
plt.show()
Bird Sightings accounted for observers over Time Graph
Observers over time
t = df.drop_duplicates(['Location']).set_index('Time')[['Location']].groupby(pd.TimeGrouper(freq='10Min')).count()
Bird Sightings divided by observers over time
a = df.loc[(df['NBirdSpecies']=='JAVAN MYNA')].set_index('Time')[['NBirdSpecies']].groupby(pd.TimeGrouper(freq='10Min')).count()

an = a.reindex(t.index).div(t.values).replace([np.inf, -np.inf], np.nan).fillna(0)

b = df.loc[(df['NBirdSpecies']=='YELLOW-VENTED BULBUL')].set_index('Time')[['NBirdSpecies']].groupby(pd.TimeGrouper(freq='10Min')).count()

bn=b.reindex(t.index).div(t.values).replace([np.inf, -np.inf], np.nan).fillna(0)

c = df.loc[(df['NBirdSpecies']=='OLIVE-BACKED SUNBIRD')].set_index('Time')[['NBirdSpecies']].groupby(pd.TimeGrouper(freq='10Min')).count()

cn = c.reindex(t.index).div(t.values).replace([np.inf, -np.inf], np.nan).fillna(0)

d = df.loc[(df['NBirdSpecies']=='BLACK-NAPED ORIOLE')].set_index('Time')[['NBirdSpecies']].groupby(pd.TimeGrouper(freq='10Min')).count()

dn = d.reindex(t.index).div(t.values).replace([np.inf, -np.inf], np.nan).fillna(0)

e = df.loc[(df['NBirdSpecies']=='ROCK PIGEON')].set_index('Time')[['NBirdSpecies']].groupby(pd.TimeGrouper(freq='10Min')).count()

en = e.reindex(t.index).div(t.values).replace([np.inf, -np.inf], np.nan).fillna(0)
Graph
pol = an.plot()
bn.plot(ax=pol)
cn.plot(ax=pol)
en.plot(ax=pol)
dn.plot(ax=pol)

pol.legend(["Number of Javan Myna","Number of Yellow-vented Bulbul","Number of Olive-backed Sunbird","Number of Black=naped Oriole","Number of Rock Pigeon"])
pol.grid()
plt.tight_layout()
plt.show()
