# Preresquisites

## Sources

https://www.splunk.com/en_us/blog/security/threat-hunting-for-dictionary-dga-with-peak.html

## Build a dataset

Dataset composed of:

- 1 million legitimate domains (download from Alexa 1M:  https://github.com/mozilla/cipherscan/blob/master/top1m/top-1m.csv)
- 600K entries of malicious dict-based DGA domains (100K for each category in pizd, suppobox, nymaim, big viktor, gozi, and matsnu)

Build the malicious domains list:

https://github.com/baderj/domain_generation_algorithms/tree/master

### pizd
Removed from github. Not able to process this list.

### suppobox

```sh
wget https://raw.githubusercontent.com/baderj/domain_generation_algorithms/refs/heads/master/suppobox/words1.txt
wget https://raw.githubusercontent.com/baderj/domain_generation_algorithms/refs/heads/master/suppobox/words2.txt
wget https://raw.githubusercontent.com/baderj/domain_generation_algorithms/refs/heads/master/suppobox/words3.txt
wget https://raw.githubusercontent.com/baderj/domain_generation_algorithms/refs/heads/master/suppobox/dga.py
cat words1.txt words2.txt words3.txt > allwords.txt
```

Modify dga.py to run 10,000 times instead of 85 (default value) and to take allwords.txt as input instead of words#.txt.

### nymaim2

```sh
wget https://raw.githubusercontent.com/baderj/domain_generation_algorithms/refs/heads/master/nymaim2/dga.py
wget https://raw.githubusercontent.com/baderj/domain_generation_algorithms/refs/heads/master/nymaim2/iptransformation.py
wget https://raw.githubusercontent.com/baderj/domain_generation_algorithms/refs/heads/master/nymaim2/words.json
```

Modify daydelta in order to increase the number of occurrences (100000/64).

```
diff dga.py dga_modified.py 
28c28
<     daydelta = 10
---
>     daydelta = 1563
```

Truncate the dataset to the first 100,000 entries.

### big viktor

Removed from the list

### gozi

```sh
wget https://raw.githubusercontent.com/baderj/domain_generation_algorithms/refs/heads/master/gozi/dga.py
wget https://raw.githubusercontent.com/baderj/domain_generation_algorithms/refs/heads/master/gozi/gpl
wget https://raw.githubusercontent.com/baderj/domain_generation_algorithms/refs/heads/master/gozi/luther
wget https://raw.githubusercontent.com/baderj/domain_generation_algorithms/refs/heads/master/gozi/nasa
wget https://raw.githubusercontent.com/baderj/domain_generation_algorithms/refs/heads/master/gozi/rfc4343
```

Modify the value in the range from `12` to `100000` to increase the number of occurrences in the loop.

```sh
diff dga.py dga_modified.py 
35c35
<     for i in range(12):
---
>     for i in range(100000):
```

### matsnu

Removed from the list.

### Dataset consolidation

Now, let's build the full dataset.

```sh
cat gozi/dataset_gozi.txt nymaim2/dataset_nymaim2.txt suppobox/dataset_suppobox.txt > dataset.txt
```

We want to append ",1" at the end of each line.

```sh
sed 's/$/,1/' dataset.txt > dataset_malicious.txt
```

Let's take the first 300,000 entries from Alexa1M to balance with the malicious domains.

```sh
cut -d "," -f2 top-1m.csv | head -n 300000 > dataset.txt
sed 's/$/,0/' dataset.txt > dataset_legit.txt
```

Now let's compile the final CSV file:

```sh
echo "domain,label" > final_dataset.csv
cat dataset_malicious.txt dataset_legit.txt >> final_dataset.csv
```

Let's check:

```sh
head final_dataset.csv 
domain,label
potemquinuchrisdechrsua.com,1
vociferanturnec.com,1
magnumquodinteriorem.com,1
apostlibraryp.com,1
ppertheologispurgatorio.com,1
campanaasciisimulmay.com,1
centieshoestnusudicitur.com,1
ipsiuspapasicutsanctumsi.com,1
vitaveniassimiserieuang.com,1
```

```sh
tail final_dataset.csv 
showbizjobs.com,0
biaojunart.com,0
chandamama.com,0
beg.wroclaw.pl,0
massrealty.com,0
xxking.com,0
labhoro.com,0
yassination.com,0
googleenterprise.blogspot.in,0
pestawirausaha.com,0
```

