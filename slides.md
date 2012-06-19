# mdma-slides

!SLIDE

# **m**eta**d**ata **ma**nagement *in four iterations*

}}} images/meta.jpg

!SLIDE left

# Goals for FINC mdma

* handle **various** data sources

    *BSZ, Naxos, E-Book Library, Nutzergesteuerte Erwerbung (PDA), ...*

* export **FINCMARC** as deliverable for VuFind **frontend**
* handle **deduplication**, **Libero** item data, MARC field rewrites, ...

!SLIDE left

# 1st iteration ↺

* PHP (261) and XML-driven (89) configuration
* leveraging common utils: *mv, yaz-marcdump, iconv*
* **bootstrapped** the metadata management

``` xml
<source file="KA_010_1994-tit.mrc">
    <prefix tag="001" prefix="PPN000000_" /> 
    <mergeaut />
</source>

<source file="NaxosNML-full.mrc">
    <iconv from="iso8859-1" />
    <copytag totag="001" fromtag="028a" />
    <prefix tag="001" prefix="EXT000005_" />
    <tagreplace tag="856u"
        pattern="^.*(=[^=]+$)" 
        replace="http://example.com/sc.asp?ver=2.0&amp;item_code$1"/>
</source>
```

``` php
$filelist = glob("$tdir/$workfix*");
    echo "processing " . count($filelist) . " chunks\n";
    foreach($filelist as $file) {
        foreach($source->children() as $action) {
            ...
```


!SLIDE left

# 2nd iteration ↺

* Python (1814) and XML-driven (148) configuration
* libraries: *pymarc, libxml, sqlalchemy*
* **NEW!** metadata snapshot stored in **database** (MySQL)

``` xml
<datasource kind="title" source_id="0" record_field="001"
    content_type="application/marc" location="/var/i/000/2*/*-tit.mrc">
    <process>
        <fm_cloneOriginal />
        <fm_setField field="001" value="finc_id()" />
        <bag_getKeyFromOriginal field="020" subfield="a" key="isbn" />
        <fm_setLocalInfo />
        <bag_getLiberoItemData />
    </process>
</datasource>
```

``` python
for j, record in enumerate(record_iterator, start=1):
    ...
    for command in _commands:
        if 'skip_this' not in bag:
            bag = command.execute(bag)
    ...
```

!SLIDE left

# **slow** on 9.1GB of data and 19M records (and that's just one data source)

}}} images/slow.jpg


!SLIDE left

# 2nd iteration slowness

* needed to reach out to Libero for **every** record
* no multiprocessing / threading
* a bit of ORM overhead


!SLIDE left

# 3rd iteration ↺

* 2nd iteration +
    * multiprocessing with 10–200 worker
    * plain SQL
    * optimized (batched) Libero-requests
    * a bit of memcached
* Python (1760), XML (451)

``` python
QUEUE_SIZE = 20000
tasks = multiprocessing.JoinableQueue(QUEUE_SIZE)
num_workers = args.workers or multiprocessing.cpu_count() * 40
workers = [ Worker(tasks, appconfig, iconf) for i in range(num_workers) ]
_ = [ w.start() for w in workers ]
...
with open(filename, 'r') as handle:
    reader = pymarc.MARCReader(handle.read(), to_unicode=True)
    for i, record in enumerate(reader):
        tasks.put(record)
```


!SLIDE left

# 3rd iteration: faster, but still sequential – still more control over data needed

}}} images/tape.gif



!SLIDE left

# 4th iteration ↺

* shift to a slightly different processing model
* **NEW!** **cache** all Libero item data (MySQL) – less network overhead – not possible without Libero *hacks*
* **NEW!** hold all raw source data in **elasticsearch** – faster, more fine grained access
* no more XML configuration, but one (or more) python scripts per data source
* Python (1708), Java (1292), bash (800)

``` python
isbns = set()
for subfield in doc['content']['020']:
    if 'a' in subfield:
        # 020.a may look like: '9780948838903 (pbk.) :'
        isbns.add(subfield['a'].split()[0])

for isbn in isbns:
    # dedup here
```


!SLIDE left

# 4th iteration data access

* flexible ad-hoc queries on MARC
* all records with ISBN 9781430272304

``` sh
$ curl -XGET 0.0.0.0:9200/_search?q=content.020.a:9781430272304
```

* kinds of classifications on NEP

``` sh
$ curl -XPOST "http://0.0.0.0:9200/nep/_search?pretty=true" -d
    '{ "query" : { "match_all" : {} }, 
       "facets" : { "tags" : { "terms" : { "field" : "content.072.2" }}}}'
```

* pipeable processing (streams records from elasticsearch via processing script into metadata db)

``` sh
$ ./scroll.sh --index bsz --meta.kind=title | ./scripts/bsz.py
```


!SLIDE left

# 4th iteration summary

* fast (~10x speedup with fewer workers vs 2nd iteration)
* flexible data access, easier debugging

**but ...**

* data redundancy (files and index)
* no more declarative XML configuration

!SLIDE left

# lessons learned so far

* **cache** everything (libero data), never go over the network for data you'll access often, stale data (few hours) is ok here
* make data item **accessible** (elasticsearch for currently 7 sources, more to come soon) / within milliseconds (still needs some work)
* assume your data-flow can **change** tomorrow (new deduplication workflow, new source, new fields, etc.)


!SLIDE left

# FIN

}}} images/Nakonechna.jpg