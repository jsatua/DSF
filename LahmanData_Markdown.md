---
title: "Lahman Dataset"
author: "J Stahl"
date: "July 7, 2017"
output:
  html_document: default
  word_document: default
---
Baseball: Hall of Fame
(Lahman Dataset)

```
The Lahman Baseball Database contains individual and team statistics from 1871 to present, and has been available as a free download since 1995. <http://seanlahman.com/baseball-archive/statistics/>
```
================

#### Load the libraries

    librar(purrr)
    library(tidyr)
    library(dplyr)

##### Find the distinct players that appear in HallOfFame
```
nominated <- HallOfFame %>% 
distinct(playerID)

nominated %>%
```
#### Count the number of players in nominated
```
count()

nominated_full <- nominated %>%
```
#### Join to Master
```
left_join(Master, by = "playerID") %>%
```
#### Return playerID, nameFirst, nameLast
```
select(playerID, nameFirst, nameLast)
```
### There were 1,239 nominees for the Hall of Fame. 
### *So, how selective is the Hall of Fame?*

#### Find distinct players in HallOfFame
```
inducted <- HallOfFame %>% 
  filter(inducted == "Y") %>%
  distinct(playerID)

inducted %>% 
```
#### Count the number of players in inducted
```
count(playerID)

inducted_full <- inducted %>% 
```

#### Join to Master
```
left_join(Master, by = "playerID") %>% 
```

#### Return playerID, nameFirst, nameLast
```
select(playerID, nameFirst, nameLast)
```

### 312 players have been inducted into the Hall of Fame out of 1,239 nominees
### *What separates the nominees who were inducted from the nominees who were not?*
### *Did nominees who were inducted earn more awards than nominees who were not inducted?*

#### Tally the number of awards in AwardsPlayers by playerID
```
nAwards <- AwardsPlayers %>% 
  group_by(playerID) %>% 
  tally()

nAwards %>% 
```
#### Filter to just the players in inducted 
```
  semi_join(inducted, by = "playerID") %>% 
```
#### Calculate the mean number of awards per player
```
  summarize(avg_n = mean(n, na.rm = TRUE))

nAwards %>% 
```
#### Filter to just the players in nominated 
```
  semi_join(nominated, by = "playerID") %>%
```

#### Filter to players NOT in inducted 
```
  anti_join(inducted, by = "playerID") %>%
```

#### Calculate the mean number of awards per player
```
  summarize(avg_n = mean(n, na.rm = TRUE))
```

### On average, inductees had 11.95 - 4.23 = 7.72 more awards than non-inductees. 
### Another possiblitiliy is that salary differentiates inductees from non-inductees. 
### *Is the maximum salary earned by inductees greater than the maximum salary earned by nominees who were not inducted?*

#### Find the players who are in nominated, but not inducted
```
notInducted <- nominated %>% 
  setdiff(inducted)

Salaries %>% 
```
#### Find the players who are in notInducted
```
  semi_join(notInducted, by = "playerID") %>%
```

#### Calculate the max salary by player
```
  group_by(playerID) %>% 
  summarize(max_salary = max(salary, na.rm = TRUE)) %>% 
```

#### Calculate the average of the max salaries
```
  summarize(avg_salary = mean(max_salary, na.rm = TRUE))
```
#### Repeat for players who were inducted
```
Salaries %>% 
  semi_join(inducted, by = "playerID") %>% 
  group_by(playerID) %>% 
  summarize(max_salary = max(salary, na.rm = TRUE)) %>% 
  summarize(avg_salary = mean(max_salary, na.rm = TRUE))
```

### The average salary of players who were inducted was $5,079,720 - $4,677,737 = $401,983 more per year.
### So, the inducted players won more awards and had an higher average salary than nominees who were not inducted.
### *Is there anything else to consider with this dataset (besides the NA values we removed)-*
### *How about checking to account for induction five years after retirement, which is a rule?*
```
  Appearances %>% 
```

#### Filter Appearances against nominated
#### Find last year played by player
```
  group_by(playerID) %>% 
  summarize(last_year = max(yearID)) %>% 
```

#### Join to full HallOfFame
```
  left_join(HallOfFame, by = "playerID") %>% 
```

#### Filter for unusual observations
```
  filter(last_year >= yearID)```
```

### Quite a few players were nominated before they retired, but less frequently in recent years.
### *This is a good point to start working on a model using the factors identified from this exploratory analysis.*