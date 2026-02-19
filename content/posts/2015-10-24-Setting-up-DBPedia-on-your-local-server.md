---
title: "Setting up DBPedia Spotlight on your local server"
description: "DBPedia Spotlight on your local server"
tags: [dbpedia, ubuntu, unix, nlp]
comments: true
share: true
cover_image: '/content/images/2014/11/materialdesign-goals-swirlanddot_large_xhdpi.jpeg'
---

## Intro : 

DBPedia Spotlight can be queried over the API which they provide. But for our convenience, it is not always possible to do so. 

So setting it up locally was the best solution.

You can run DBpedia Spotlight from the comfort of your own machine in one or many of the following ways:

- **Web Service** (no installation needed). We offer a WADL service descriptor, so with Eclipse or Netbeans you can automagically create a client to call our Web Service. See: Web Service
- **JAR**. We offer a jar with all dependencies included. You can just download it and run from command line. See: Run from a Jar
- **Maven**. Our build is mavenized, which means that you can use the scala plugin to run our classes from command line. See: Build from Source with Maven
- **Ubuntu/Debian package**. We are also starting to share our downloads as debian packages so that anybody can just install DBpedia Spotlight directly from apt-get, Synaptic or their favorite package manager. See:Debian-Package-Installation:-How-To
- **WAR Files/ Tomcat**. DBpedia Spotlight is also build as a WAR file. You can use it through Apache Tomcat.

We would be doing it the **JAR** way.


## Get set. Go

Requirements

- Java 1.6+
- RAM of appropriate size for the spotter lexicon you need

First we will install a pre-packaged lightweight deployment to get you started.

**Lucene**

{% highlight bash %}
sys2@sys2:~$ wget http://spotlight.dbpedia.org/download/release-0.6/dbpedia-spotlight-quickstart-0.6.5.zip
--2015-10-22 10:28:16--  http://spotlight.dbpedia.org/download/release-0.6/dbpedia-spotlight-quickstart-0.6.5.zip
Resolving spotlight.dbpedia.org (spotlight.dbpedia.org)... 134.155.95.15
Connecting to spotlight.dbpedia.org (spotlight.dbpedia.org)|134.155.95.15|:80... connected.
HTTP request sent, awaiting response... 302 Found
Location: http://wifo5-04.informatik.uni-mannheim.de/downloads/release-0.6/dbpedia-spotlight-quickstart-0.6.5.zip [following]
--2015-10-22 10:28:17--  http://wifo5-04.informatik.uni-mannheim.de/downloads/release-0.6/dbpedia-spotlight-quickstart-0.6.5.zip
Resolving wifo5-04.informatik.uni-mannheim.de (wifo5-04.informatik.uni-mannheim.de)... 134.155.95.17
Connecting to wifo5-04.informatik.uni-mannheim.de (wifo5-04.informatik.uni-mannheim.de)|134.155.95.17|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 156569414 (149M) [application/zip]
Saving to: ‘dbpedia-spotlight-quickstart-0.6.5.zip’

100%[====================================================================================================>] 15,65,69,414 1.14MB/s   in 5m 51s 

2015-10-22 10:34:09 (435 KB/s) - ‘dbpedia-spotlight-quickstart-0.6.5.zip’ saved [156569414/156569414]
sys2@sys2:~$
sys2@sys2:~$ unzip dbpedia-spotlight-quickstart-0.6.5.zip
Archive:  dbpedia-spotlight-quickstart-0.6.5.zip
   creating: dbpedia-spotlight-quickstart-0.6.5/data/
   creating: dbpedia-spotlight-quickstart-0.6.5/data/index/
  inflating: dbpedia-spotlight-quickstart-0.6.5/data/index/_1.cfs  
  inflating: dbpedia-spotlight-quickstart-0.6.5/data/index/_2.cfs  
  inflating: dbpedia-spotlight-quickstart-0.6.5/data/index/_3.cfs  
  inflating: dbpedia-spotlight-quickstart-0.6.5/data/index/segments.gen  
  inflating: dbpedia-spotlight-quickstart-0.6.5/data/index/segments_5  
  inflating: dbpedia-spotlight-quickstart-0.6.5/data/index/similarity-thresholds.txt  
   creating: dbpedia-spotlight-quickstart-0.6.5/data/opennlp/
   creating: dbpedia-spotlight-quickstart-0.6.5/data/opennlp/english/
  inflating: dbpedia-spotlight-quickstart-0.6.5/data/opennlp/english/en-chunker.bin  
 extracting: dbpedia-spotlight-quickstart-0.6.5/data/opennlp/english/en-chunker.zip  
  inflating: dbpedia-spotlight-quickstart-0.6.5/data/opennlp/english/en-ner-location.bin  
 extracting: dbpedia-spotlight-quickstart-0.6.5/data/opennlp/english/en-ner-location.zip  
  inflating: dbpedia-spotlight-quickstart-0.6.5/data/opennlp/english/en-ner-organization.bin  
 extracting: dbpedia-spotlight-quickstart-0.6.5/data/opennlp/english/en-ner-organization.zip  
  inflating: dbpedia-spotlight-quickstart-0.6.5/data/opennlp/english/en-ner-person.bin  
 extracting: dbpedia-spotlight-quickstart-0.6.5/data/opennlp/english/en-ner-person.zip  
  inflating: dbpedia-spotlight-quickstart-0.6.5/data/opennlp/english/en-pos-maxent.bin  
 extracting: dbpedia-spotlight-quickstart-0.6.5/data/opennlp/english/en-pos-maxent.zip  
  inflating: dbpedia-spotlight-quickstart-0.6.5/data/opennlp/english/en-sent.bin  
 extracting: dbpedia-spotlight-quickstart-0.6.5/data/opennlp/english/en-sent.zip  
  inflating: dbpedia-spotlight-quickstart-0.6.5/data/opennlp/english/en-token.bin  
 extracting: dbpedia-spotlight-quickstart-0.6.5/data/opennlp/english/en-token.zip  
  inflating: dbpedia-spotlight-quickstart-0.6.5/data/opennlp/README.txt  
  inflating: dbpedia-spotlight-quickstart-0.6.5/data/pos-en-general-brown.HiddenMarkovModel  
  inflating: dbpedia-spotlight-quickstart-0.6.5/data/spotter.dict  
  inflating: dbpedia-spotlight-quickstart-0.6.5/dbpedia-spotlight-0.6.5-jar-with-dependencies.jar  
  inflating: dbpedia-spotlight-quickstart-0.6.5/run.sh  
  inflating: dbpedia-spotlight-quickstart-0.6.5/server.properties  
  inflating: dbpedia-spotlight-quickstart-0.6.5/apache-2.0.txt  
  inflating: dbpedia-spotlight-quickstart-0.6.5/lingpipe-license-1.txt  
sys2@sys2:~$ cd dbpedia-spotlight-quickstart-0.6.5/
sys2@sys2:~/dbpedia-spotlight-quickstart-0.6.5$ 
{% endhighlight %}

**Statistical**

{% highlight bash %}
sys2@sys2:~$ wget http://spotlight.sztaki.hu/downloads/version-0.1/en.tar.gz
--2015-10-22 11:12:58--  http://spotlight.sztaki.hu/downloads/version-0.1/en.tar.gz
Resolving spotlight.sztaki.hu (spotlight.sztaki.hu)... 193.225.89.3
Connecting to spotlight.sztaki.hu (spotlight.sztaki.hu)|193.225.89.3|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 2257455959 (2.1G) [application/x-gzip]
Saving to: ‘en.tar.gz’

 100%[==================================================================================================>] 2,25,74,55,959  935KB/s   in 44m 59s

2015-10-22 11:57:58 (817 KB/s) - ‘en.tar.gz’ saved [2257455959/2257455959]
sys2@sys2:~$ 
sys2@sys2:~$ wget http://spotlight.sztaki.hu/downloads/version-0.1/dbpedia-spotlight.jar
--2015-10-22 12:13:39--  http://spotlight.sztaki.hu/downloads/version-0.1/dbpedia-spotlight.jar
Resolving spotlight.sztaki.hu (spotlight.sztaki.hu)... 193.225.89.3
Connecting to spotlight.sztaki.hu (spotlight.sztaki.hu)|193.225.89.3|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 116325524 (111M) [application/java-archive]
Saving to: ‘dbpedia-spotlight.jar’

 100%[====================================================================================================>] 11,63,25,524  415KB/s   in 3m 27s 

2015-10-22 12:17:07 (549 KB/s) - ‘dbpedia-spotlight.jar’ saved [116325524/116325524]

sys2@sys2:~$ tar xvf en.tar.gz 
en/
en/model/
en/model/res.mem
en/model/res.mem_
en/model/tokens.mem
en/model/context.mem
en/model/sf.mem
en/model/candmap.mem
en/model.properties
en/stopwords.list
en/spotter_thresholds.txt
en/opennlp/
en/opennlp/pos-maxent.bin
en/opennlp/token.bin
en/opennlp/chunker.bin
en/opennlp/sent.bin
sys2@sys2:~$
sys2@sys2:~$

{% endhighlight %}

## Add the Data corpus

We can run the model right now, but I will do so after adding the larger data corpus.


{% highlight bash %}
sys2@sys2:~/dbpedia-spotlight-quickstart-0.6.5$ cd data
sys2@sys2:~/dbpedia-spotlight-quickstart-0.6.5/data$ wget http://spotlight.dbpedia.org/download/release-0.5/context-index-compact.tgz
--2015-10-22 12:25:58--  http://spotlight.dbpedia.org/download/release-0.5/context-index-compact.tgz
Resolving spotlight.dbpedia.org (spotlight.dbpedia.org)... 134.155.95.15
Connecting to spotlight.dbpedia.org (spotlight.dbpedia.org)|134.155.95.15|:80... connected.
HTTP request sent, awaiting response... 302 Found
Location: http://wifo5-04.informatik.uni-mannheim.de/downloads/release-0.5/context-index-compact.tgz [following]
--2015-10-22 12:25:59--  http://wifo5-04.informatik.uni-mannheim.de/downloads/release-0.5/context-index-compact.tgz
Resolving wifo5-04.informatik.uni-mannheim.de (wifo5-04.informatik.uni-mannheim.de)... 134.155.95.17
Connecting to wifo5-04.informatik.uni-mannheim.de (wifo5-04.informatik.uni-mannheim.de)|134.155.95.17|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 12017976481 (11G) [application/x-gzip]
Saving to: ‘context-index-compact.tgz’


 100%[=================================================================================================>] 12,01,79,76,481 1.16MB/s   in 4h 10m 

2015-10-22 16:36:37 (781 KB/s) - ‘context-index-compact.tgz’ saved [12017976481/12017976481]

sys2@sys2:~/dbpedia-spotlight-quickstart-0.6.5/data$ tar zxvf context-index-compact.tgz
index-withSF-withTypes-compressed/
index-withSF-withTypes-compressed/_at.cfs
index-withSF-withTypes-compressed/_66.cfs
index-withSF-withTypes-compressed/segments_9t
index-withSF-withTypes-compressed/_99.cfs
index-withSF-withTypes-compressed/similarity-thresholds.txt
index-withSF-withTypes-compressed/segments.gen
index-withSF-withTypes-compressed/_33.cfs
sys2@sys2:~/dbpedia-spotlight-quickstart-0.6.5/data$ mv index-withSF-withTypes-compressed index
sys2@sys2:~/dbpedia-spotlight-quickstart-0.6.5/data$ wget http://spotlight.dbpedia.org/download/release-0.4/surface_forms-Wikipedia-TitRedDis.uriThresh75.tsv.spotterDictionary.gz
sys2@sys2:~/dbpedia-spotlight-quickstart-0.6.5/data$ gunzip surface_forms-Wikipedia-TitRedDis.uriThresh75.tsv.spotterDictionary.gz
sys2@sys2:~/dbpedia-spotlight-quickstart-0.6.5/data$ mv surface_forms-Wikipedia-TitRedDis.uriThresh75.tsv.spotterDictionary spotter.dict
sys2@sys2:~/dbpedia-spotlight-quickstart-0.6.5/data$ 
{% endhighlight %}


## Testing the installation :

In order to test the installation, do a

{% highlight bash %}
sys2@sys2:~/dbpedia-spotlight-quickstart-0.6.5/data$ curl http://sys2:2222/rest/annotate   -H "Accept: text/xml"   --data-urlencode "text=The earliest authenticated human remains in South Asia date to about 30,000 years ago.[26] Nearly contemporaneous Mesolithic rock art sites have been found in many parts of the Indian subcontinent, including at the Bhimbetka rock shelters in Madhya Pradesh.[27] Around 7000 BCE, the first known Neolithic settlements appeared on the subcontinent in Mehrgarh and other sites in western Pakistan.[28] These gradually developed into the Indus Valley Civilisation,[29] the first urban culture in South Asia;[30] it flourished during 2500–1900 BCE in Pakistan and western India along the river valleys of Indus and Sarasvati.[31] Centred on cities such as Mohenjo-daro, Harappa, Rakhigarhi, Dholavira, and Kalibangan, and relying on varied forms of subsistence, the civilisation engaged robustly in crafts production and wide-ranging trade."   --data "confidence=0"   --data "support=0"


sys2@sys2:~/dbpedia-spotlight-quickstart-0.6.5/data$ curl http://sys2:2222/rest/annotate   -H "Accept: text/xml"   --data-urlencode "text=The earliest authenticated human remains in South Asia date to about 30,000 years ago.[26] Nearly contemporaneous Mesolithic rock art sites have been found in many parts of the Indian subcontinent, including at the Bhimbetka rock shelters in Madhya Pradesh.[27] Around 7000 BCE, the first known Neolithic settlements appeared on the subcontinent in Mehrgarh and other sites in western Pakistan.[28] These gradually developed into the Indus Valley Civilisation,[29] the first urban culture in South Asia;[30] it flourished during 2500–1900 BCE in Pakistan and western India along the river valleys of Indus and Sarasvati.[31] Centred on cities such as Mohenjo-daro, Harappa, Rakhigarhi, Dholavira, and Kalibangan, and relying on varied forms of subsistence, the civilisation engaged robustly in crafts production and wide-ranging trade."   --data "confidence=0"   --data "support=0"

<?xml version="1.0" encoding="utf-8"?>

<Annotation text="The earliest authenticated human remains in South Asia date to about 30,000 years ago.[26] Nearly contemporaneous Mesolithic rock art sites have been found in many parts of the Indian subcontinent, including at the Bhimbetka rock shelters in Madhya Pradesh.[27] Around 7000 BCE, the first known Neolithic settlements appeared on the subcontinent in Mehrgarh and other sites in western Pakistan.[28] These gradually developed into the Indus Valley Civilisation,[29] the first urban culture in South Asia;[30] it flourished during 2500–1900 BCE in Pakistan and western India along the river valleys of Indus and Sarasvati.[31] Centred on cities such as Mohenjo-daro, Harappa, Rakhigarhi, Dholavira, and Kalibangan, and relying on varied forms of subsistence, the civilisation engaged robustly in crafts production and wide-ranging trade." confidence="0.0" support="0" types="" sparql="" policy="whitelist">

<Resources>

<Resource URI="http://dbpedia.org/resource/South_Asia" support="2850" types="Freebase:/book/book_subject,Freebase:/book,Freebase:/location/location,Freebase:/location,Freebase:/organization/organization_scope,Freebase:/organization,Freebase:/location/region,Freebase:/people/ethnicity,Freebase:/people" surfaceForm="South Asia" offset="44" similarityScore="0.13776831328868866" percentageOfSecondRank="-1.0"/>

<Resource URI="http://dbpedia.org/resource/Mesolithic" support="681" types="" surfaceForm="Mesolithic" offset="114" similarityScore="0.13642436265945435" percentageOfSecondRank="-1.0"/>

<Resource URI="http://dbpedia.org/resource/Indian_subcontinent" support="1497" types="DBpedia:Continent,DBpedia:PopulatedPlace,DBpedia:Place,Schema:Place,Schema:Continent,Freebase:/location/region,Freebase:/location,Freebase:/location/location" surfaceForm="Indian subcontinent" offset="177" similarityScore="0.141897514462471" percentageOfSecondRank="-1.0"/>

<Resource URI="http://dbpedia.org/resource/Madhya_Pradesh" support="2950" types="DBpedia:Settlement,DBpedia:PopulatedPlace,DBpedia:Place,Schema:Place,Freebase:/location/in_state,Freebase:/location,Freebase:/location/statistical_region,Freebase:/location/dated_location,Freebase:/location/location,Freebase:/book/author,Freebase:/book,Freebase:/location/administrative_division" surfaceForm="Madhya Pradesh" offset="242" similarityScore="0.13563162088394165" percentageOfSecondRank="-1.0"/>

<Resource URI="http://dbpedia.org/resource/Common_Era" support="1247" types="" surfaceForm="BCE" offset="274" similarityScore="0.1675749570131302" percentageOfSecondRank="0.20302364934710979"/>

<Resource URI="http://dbpedia.org/resource/Neolithic" support="2903" types="" surfaceForm="Neolithic" offset="295" similarityScore="0.11610618233680725" percentageOfSecondRank="-1.0"/>

<Resource URI="http://dbpedia.org/resource/Pakistan" support="23561" types="DBpedia:Country,DBpedia:PopulatedPlace,DBpedia:Place,Schema:Place,Schema:Country,Freebase:/location/country,Freebase:/location,Freebase:/organization/organization_member,Freebase:/organization,Freebase:/biology/breed_origin,Freebase:/biology,Freebase:/location/statistical_region,Freebase:/military/military_combatant,Freebase:/military,Freebase:/people/ethnicity,Freebase:/people,Freebase:/location/dated_location,Freebase:/law/court_jurisdiction_area,Freebase:/law,Freebase:/sports/sport_country,Freebase:/sports,Freebase:/government/governmental_jurisdiction,Freebase:/government,Freebase:/olympics/olympic_participating_country,Freebase:/olympics,Freebase:/location/location,Freebase:/meteorology/cyclone_affected_area,Freebase:/meteorology,Freebase:/book/book_subject,Freebase:/book,Freebase:/sports/sports_team_location" surfaceForm="Pakistan" offset="385" similarityScore="0.0921529158949852" percentageOfSecondRank="0.6893670279187828"/>

<Resource URI="http://dbpedia.org/resource/Indus_Valley_Civilization" support="424" types="Freebase:/time/event,Freebase:/time,Freebase:/book/book_subject,Freebase:/book" surfaceForm="Indus Valley Civilisation" offset="434" similarityScore="0.20086094737052917" percentageOfSecondRank="-1.0"/>

<Resource URI="http://dbpedia.org/resource/South_Asia" support="2850" types="Freebase:/book/book_subject,Freebase:/book,Freebase:/location/location,Freebase:/location,Freebase:/organization/organization_scope,Freebase:/organization,Freebase:/location/region,Freebase:/people/ethnicity,Freebase:/people" surfaceForm="South Asia" offset="492" similarityScore="0.13776831328868866" percentageOfSecondRank="-1.0"/>

<Resource URI="http://dbpedia.org/resource/Common_Era" support="1247" types="" surfaceForm="BCE" offset="539" similarityScore="0.1675749570131302" percentageOfSecondRank="0.20302364934710979"/>

<Resource URI="http://dbpedia.org/resource/Pakistan" support="23561" types="DBpedia:Country,DBpedia:PopulatedPlace,DBpedia:Place,Schema:Place,Schema:Country,Freebase:/location/country,Freebase:/location,Freebase:/organization/organization_member,Freebase:/organization,Freebase:/biology/breed_origin,Freebase:/biology,Freebase:/location/statistical_region,Freebase:/military/military_combatant,Freebase:/military,Freebase:/people/ethnicity,Freebase:/people,Freebase:/location/dated_location,Freebase:/law/court_jurisdiction_area,Freebase:/law,Freebase:/sports/sport_country,Freebase:/sports,Freebase:/government/governmental_jurisdiction,Freebase:/government,Freebase:/olympics/olympic_participating_country,Freebase:/olympics,Freebase:/location/location,Freebase:/meteorology/cyclone_affected_area,Freebase:/meteorology,Freebase:/book/book_subject,Freebase:/book,Freebase:/sports/sports_team_location" surfaceForm="Pakistan" offset="546" similarityScore="0.0921529158949852" percentageOfSecondRank="0.6893670279187828"/>

<Resource URI="http://dbpedia.org/resource/South_Asia" support="2850" types="Freebase:/book/book_subject,Freebase:/book,Freebase:/location/location,Freebase:/location,Freebase:/organization/organization_scope,Freebase:/organization,Freebase:/location/region,Freebase:/people/ethnicity,Freebase:/people" surfaceForm="India" offset="567" similarityScore="0.1619909405708313" percentageOfSecondRank="0.8368753166673841"/>

<Resource URI="http://dbpedia.org/resource/Indus_Valley_Civilization" support="424" types="Freebase:/time/event,Freebase:/time,Freebase:/book/book_subject,Freebase:/book" surfaceForm="Indus" offset="600" similarityScore="0.20086094737052917" percentageOfSecondRank="-1.0"/>

<Resource URI="http://dbpedia.org/resource/Sarasvati_River" support="60" types="Freebase:/geography/river,Freebase:/geography,Freebase:/location/location,Freebase:/location,Freebase:/geography/geographical_feature,Freebase:/geography/body_of_water,Freebase:/religion/deity,Freebase:/religion" surfaceForm="Sarasvati" offset="610" similarityScore="0.08038105070590973" percentageOfSecondRank="0.9255801653640217"/>

<Resource URI="http://dbpedia.org/resource/Mohenjo-daro" support="136" types="DBpedia:WorldHeritageSite,DBpedia:Place,Schema:Place,Freebase:/protected_sites/listed_site,Freebase:/protected_sites,Freebase:/location/location,Freebase:/location" surfaceForm="Mohenjo-daro" offset="651" similarityScore="0.19224941730499268" percentageOfSecondRank="-1.0"/>

<Resource URI="http://dbpedia.org/resource/Harappa" support="163" types="DBpedia:Settlement,DBpedia:PopulatedPlace,DBpedia:Place,Schema:Place,Freebase:/location/location,Freebase:/location,Freebase:/location/statistical_region,Freebase:/location/citytown,Freebase:/location/dated_location" surfaceForm="Harappa" offset="665" similarityScore="0.17811940610408783" percentageOfSecondRank="-1.0"/>

</Resources>

</Annotation>

sys2@sys2:~/dbpedia-spotlight-quickstart-0.6.5/data$
{% endhighlight %}

So there you go.

## References

- [https://github.com/dbpedia-spotlight/dbpedia-spotlight/wiki/Run-from-a-JAR](https://github.com/dbpedia-spotlight/dbpedia-spotlight/wiki/Run-from-a-JAR)

- [https://github.com/dbpedia-spotlight/dbpedia-spotlight/wiki/Installation](https://github.com/dbpedia-spotlight/dbpedia-spotlight/wiki/Installation)

- [https://dbpedia-spotlight.github.io/demo/](https://dbpedia-spotlight.github.io/demo/)

- [https://github.com/dbpedia-spotlight/dbpedia-spotlight/wiki](https://github.com/dbpedia-spotlight/dbpedia-spotlight/wiki)

Till then. Goodbye!