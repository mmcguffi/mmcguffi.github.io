---
layout: page
title: Methods
permalink: /methods/
---

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;I first constructed a manually-curated list of over 1,200 features commonly found in engineered plasmids and I created a local BLAST database of these features. I then wrote a python script which takes an inputted DNA sequence or FASTA file and BLASTs the entire sequence against curated database. Many of the hits are redundant given the nature of the manually curated list and engineered parts in general, so I filtered the results to give proper annotations. During the filtering process, I also located fragments of these engineered parts, which I define as having at least 90% identity, but less than 95% match length (these values were empirically determined to give accurate results).  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Using a database of nearly 20,000 engineered plasmid sequences provided by <a href="https://www.addgene.org/mission/">Addgene</a>, I then applied this pipeline to those plasmids, quantifying the amount of “junk” contained within these plasmids. I only analyzed CDSs, because these sequences are simple to know where they stop and start. Other sequences such as ncRNA-based origins and promoters have boundaries that are much harder to define, as often they can contain UP elements, or other features that make the start and stop positions ambiguous.

# Initialization 


```python
import glob
from Bio.Seq import Seq
from Bio import SeqIO
from Bio.SeqRecord import SeqRecord
import json
import subprocess
from Bio.SeqFeature import SeqFeature, FeatureLocation
from Bio.Alphabet import generic_dna
import time
from tempfile import NamedTemporaryFile


import plotly
plotly.offline.init_notebook_mode()
import plotly.offline as py
import plotly.graph_objs as go
```


<script type='text/javascript'>if(!window.Plotly){define('plotly', function(require, exports, module) {/**
* plotly.js v1.41.3
* Copyright 2012-2018, Plotly, Inc.
* All rights reserved.
* Licensed under the MIT license
*/



```python
def bash(inCommand, autoOutfile = True):
    if autoOutfile == True:
        tmp = NamedTemporaryFile()
        subprocess.call(inCommand +' > '+tmp.name, shell=True)
        f = open(tmp.name,'r')
        tmp.close()
        return(f.read())
    else:
        subprocess.call(inCommand, shell=True)  
def BLAST(seq, db='nr_db', BLASTtype="p", flags = 'sseqid pident qstart qend sseq slen send sstart'): 
    query = NamedTemporaryFile()
    SeqIO.write(SeqRecord(Seq(seq), id="temp"), query.name, "fasta")
    tmp = NamedTemporaryFile()
    bash(
        'blast'+BLASTtype+' -query ' + query.name + ' -out ' + tmp.name +
        ' -db /Users/mattmcguffie/database/BLAST_dbs/' + db +
        ' -max_target_seqs 20000 -outfmt "6 '+flags+'"')
    with open(tmp.name, "r") as file_handle:  #opens BLAST file
        align = file_handle.readlines()
        
    tmp.close() 
    query.close() 
    return align
```


```python
#loads all plasmids in db
with open("/Users/mattmcguffie/database/addgene-plasmids-sequences-2018-10-08.json", "r") as file_handle:
    plasmid = json.loads(file_handle.read())['plasmids'] #this takes ~5+ sec to load
#loads just the plasmids that have full seqs
fullPlasSeqList=[]
for plas in plasmid:
    if len(plas['sequences']['public_addgene_full_sequences'])==1:
        fullPlasSeqList.append(plas)
len(fullPlasSeqList)
```




    20705



# make BLAST db


```python
seqlist=[]
for infile_loc in glob.glob('/Users/mattmcguffie/database/standard_features_snapgene/*.gb'):
    with open(infile_loc, "r") as file_handle:
        opendict = SeqIO.to_dict(SeqIO.parse(file_handle, "gb"))["."]
        opendict.id=infile_loc.split("/")[-1].replace(" ","_")
        opendict.description=''
        seqlist.append(opendict)
with open("/Users/mattmcguffie/database/snapgene_feature_list.fasta", "w") as handle:
    SeqIO.write(seqlist, handle, "fasta")
```


```python
bash('makeblastdb -in /Users/mattmcguffie/database/snapgene_feature_list.fasta -out /Users/mattmcguffie/database/BLAST_dbs/snapgene_feature_list_db -dbtype nucl -parse_seqids')

```




    ['',
     '',
     'Building a new DB, current time: 10/19/2018 15:49:04',
     'New DB name:   /Users/mattmcguffie/database/BLAST_dbs/snapgene_feature_list_db',
     'New DB title:  /Users/mattmcguffie/database/snapgene_feature_list.fasta',
     'Sequence type: Nucleotide',
     'Keep MBits: T',
     'Maximum file size: 1000000000B',
     'Adding sequences from FASTA; added 1239 sequences in 0.0536189 seconds.',
     '']



# annotating pipeline


```python
#main pipline
fileloc="/Users/mattmcguffie/database/plasmids/coll.fasta"
record=list(SeqIO.parse(fileloc, "fasta"))[0]
query=str(record.seq)

hits = BLAST(query, db ="snapgene_feature_list_db",BLASTtype="n",flags='qstart qend sseqid sframe pident slen sseq')
record.name=fileloc.split("/")[-1].split(".")[0]
record.seq.alphabet=generic_dna
record.annotations["topology"] = "circular"

for hit in hits:
    splitAlign=hit.split()
    
    qstart = int(splitAlign[0])-1
    qend = int(splitAlign[1])-1
    sseqid = splitAlign[2]
    sframe = int(splitAlign[3])
    pident = float(splitAlign[4])
    slen = int(splitAlign[5])
    sseq = splitAlign[6]
    percmatch = round((len(sseq)/slen)*100,3)
        
    Type=""
    misc = ""
    
    if pident>=90: #identity of match 
        if percmatch < 95: #length of math
            Type="Fragment"
        name = sseqid.split(".gb")[0].replace("_"," ")
        if "promoter" in name:
            if Type == "Fragment":
                misc = "promoter"
            else:
                Type = "promoter"
        elif "ori" in name:
            if Type == "Fragment":
                misc = "origin"
            else:
                Type = "origin"
        elif "terminator" in name:
            if Type == "Fragment":
                misc = "terminator"
            else:
                Type = "terminator"
        elif "R" in name[1:]:
            if Type == "Fragment":
                misc = "resistance"
            else:
                Type = "resistance"
        else:
            if Type == "Fragment":
                misc = "misc"
            else:
                Type = "misc"        

        record.features.append(SeqFeature(FeatureLocation(qstart, qend), type=Type,qualifiers={"label": name,"identity":pident,"match length":percmatch,"Type: ":misc}, strand = sframe))

with open("/Users/mattmcguffie/Desktop/test2.gbk", "w") as handle:
    SeqIO.write(record, handle, "genbank")
```


```python
#print hits
seqSpace=[""]*len(query)
for hit in hits:
    splitAlign=hit.split()cx
    
    qstart = int(splitAlign[0])-1
    qend = int(splitAlign[1])-1
    sseqid = splitAlign[2]
    sframe = ixoLpoxlitAlign[3])
    pident = float(splitAlign[4])
    slen = int(splitAlign[5])
    sseq = splitAlign[6]
    percmatch = round((len(sseq)/slen)*100,3)
    print(qstart,qend)
    print(sseqid,pident,percmatch,slen,sep="\t")
    print()

```

    5034 5894
    AmpR_(3).gb	99.535	100.0	861
    
    5034 5894
    AmpR_(4).gb	99.304	100.116	861
    
    5034 5894
    AmpR.gb	98.839	100.0	861
    
    555 1358
    URA3.gb	99.876	100.0	804
    
    5035 5824
    bla(M).gb	99.747	99.371	795
    
    2722 3432
    yeGFP.gb	98.596	99.303	717
    
    2722 3432
    YPet.gb	96.91	99.72	714
    
    2722 3432
    Azurite_BFP.gb	96.218	100.0	714
    
    2722 3432
    CyPet.gb	94.25	99.86	714
    
    4275 4863
    ori.gb	99.321	100.0	589
    
    5034 5894
    AmpR_(2).gb	87.689	100.0	861
    
    1661 2116
    f1_ori_(2).gb	100.0	100.0	456
    
    1661 2116
    M13_ori_(4).gb	99.561	88.716	514
    
    4094 4638
    RSF_ori.gb	94.128	72.667	750
    
    1661 2116
    M13_ori.gb	98.684	89.412	510
    
    1661 2116
    M13_ori_(2).gb	98.684	100.885	452
    
    2724 3432
    GFP_(2).gb	86.957	99.442	717
    
    1688 2116
    f1_ori_(4).gb	100.0	100.0	429
    
    2724 3432
    superecliptic_pHluorin.gb	86.434	99.721	717
    
    2724 3432
    ecliptic_pHluorin.gb	86.294	99.721	717
    
    1661 2116
    f1_ori.gb	95.624	100.219	456
    
    2724 3432
    ratiometric_pHluorin.gb	84.734	99.582	717
    
    2724 3432
    GFPuv.gb	84.594	99.582	717
    
    2725 3432
    Cycle_3_GFP.gb	84.572	99.028	720
    
    5332 5710
    ISS.gb	99.736	100.0	379
    
    1736 2116
    M13_ori_(3).gb	99.475	100.0	381
    
    2725 3432
    superfolder_GFP_(2).gb	83.871	99.86	714
    
    1758 2116
    f1_ori_(3).gb	100.0	100.0	359
    
    3473 3722
    CYC1_terminator.gb	98.4	100.806	248
    
    4334 4878
    CloDF13_ori.gb	80.79	75.372	739
    
    338 554
    URA3_promoter_(2).gb	99.539	100.0	217
    
    338 554
    URA3_promoter_(3).gb	96.313	100.463	216
    
    338 554
    URA3_promoter_(4).gb	96.313	100.463	216
    
    3519 3710
    CYC1_terminator_(2).gb	98.438	101.053	190
    
    338 554
    URA3_promoter_(5).gb	94.931	100.0	217
    
    338 554
    URA3_promoter.gb	94.47	102.844	211
    
    4275 4863
    p15A_ori.gb	77.685	109.158	546
    
    2118 2287
    lacZ_(2).gb	98.246	5.561	3075
    
    2124 2287
    lacZ-alpha.gb	98.182	94.828	174
    
    2118 2270
    lacZ.gb	99.346	5.02	3048
    
    5895 5999
    AmpR_promoter_(2).gb	100.0	100.0	105
    
    5897 5999
    AmpR_promoter_(3).gb	100.0	100.0	103
    
    4271 4406
    ColA_ori.gb	91.241	21.541	636
    
    4000 4092
    lacI.gb	100.0	8.587	1083
    
    5895 5986
    AmpR_promoter.gb	100.0	100.0	92
    
    5895 5966
    AmpR_promoter_(4).gb	100.0	100.0	72
    
    1607 1660
    bom.gb	100.0	38.298	141
    
    137 188
    bom.gb	100.0	36.879	141
    
    2674 2717
    MCS_(2).gb	100.0	40.741	108
    
    1 35
    rop.gb	100.0	18.229	192
    



```python

```


```python

```