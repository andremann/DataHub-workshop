## Hands -On

### Preparation

Access the HUE interface from your Web browser.
- Go to Menu > Browsers > Files
- - Locate folder /project/citylabs-workshop
- - Locate __my__ folder: /project/citylabs-workshop/group-0
- - Locate __your__ folder /project/citylabs-workshop/group-X
- - - X = (1,2,3,4,5)

- Open in separate Tabs:
- - Query > HIVE (right-click, open in new tab)
- - Query > PIG (right-click, open in new tab)
- Inspect the Saved Queries Tab below the input text area. They contain the set of HIVE queries and PIG scripts mentioned in this tutorial.

### Step 1 - Compute Term Vectors
From texts to term vectors

- Input: gutenberg_docs.tsv 
- Output: gutenberg_terms.tsv 
- Task: Explore data

(Already done, as complicated and time consuming. Output file is in the ```Group_0``` folder.)

This step involves a set of NLP techniques:

- Remove common words (the, of, for, …)
- Part of Speech tagging (Verb, Noun, …)
- Stemming (going -> go)
- Abstract (12, 1.000, 20% -> <NUMBER>)

***Important: in what follows replace X with your group number!!!***

Create the HIVE table mapped to the files ```gutenberg\_docs.tsv```
```
-- 1.1 TABLE GUTENBERG_DOCS
CREATE EXTERNAL TABLE citylabs_workshop.gutenberg_docs_X
    (ID STRING, SOURCE STRING, LEN INT, CHARSET STRING, CONTENT STRING) 
ROW FORMAT DELIMITED 
FIELDS TERMINATED BY '\t' 
LINES TERMINATED BY '\n' 
STORED AS TEXTFILE 
LOCATION '/project/citylabs-workshop/group-0/gutenberg_docs.tsv'; 
```

```
-- 1.2 COUNT GUTENBERG_DOCS
SELECT COUNT(ID) FROM citylabs_workshop.gutenberg_docs_X;
```

```
-- 1.3 AVG(LEN) GUTENBERG_DOCS
SELECT AVG(LEN) FROM citylabs_workshop.gutenberg_docs_X;
```
*(Remember to replace X with your group number!!!)*
```
-- 1.4 TABLE GUTENBERG_TERMS
CREATE EXTERNAL TABLE citylabs_workshop.gutenberg_terms_X
    (DOC STRING, POSITION INT, TERM STRING) 
ROW FORMAT DELIMITED    
FIELDS TERMINATED BY  '\t' 
LINES TERMINATED BY '\n' 
STORED AS TEXTFILE 
LOCATION '/project/citylabs-workshop/group-0/gutenberg_terms.tsv'; 
```

```
-- 1.5 COUNT GUTENBERG_TERMS
SELECT COUNT(TERM) FROM citylabs_workshop.gutenberg_terms_X;
```

```
-- 1.6 ORDER By LENGTH GUTENBERG_TERMS
SELECT DOC, COUNT(TERM) AS LENGTH FROM citylabs_workshop.gutenberg_terms_X
GROUP BY DOC ORDER BY LENGTH DESC;
```

Lookup book id Gutenberg-11800 as follows:

http://www.gutenberg.org/ebooks/2981

Try 5307, 19116, 1, 1234.

### Step 2 - Compute Terms Frequency (TF) 

a) From term word vectors to word counts 

b) Generate doc size from word counts

c) Compute term frequency as count/len

Task: run PIG script to produce the output file:

*(Remember to replace X with your group number!!!)*

*Check PIG  Hadoop properties: mapreduce.job.queuename=/root.project/citylabs-workshop*
```
/* Step 2 - Compute Terms Frequency (TF) */
/* NB: Remember to replace X with your group number (last line) */
set mapreduce.map.memory.mb 2048 
set mapreduce.reduce.memory.mb 5120 
source = LOAD '/project/citylabs-workshop/group-0/gutenberg_terms.tsv' AS (doc_id:chararray, position:int, word:chararray); 
collection = FILTER source BY (SUBSTRING(word, 0, 1) != '<');
doc_words       = GROUP collection BY (doc_id, word);
doc_word_counts = FOREACH doc_words 
                                 GENERATE 
                                    FLATTEN(group) AS (doc_id, word), 
                                    COUNT(collection) AS num_doc_wrd_usages
                    ;

usage_bag    = GROUP doc_word_counts BY doc_id;
usage_bag_ds = FOREACH usage_bag 
      GENERATE 
group AS doc_id,
    FLATTEN 
(doc_word_counts.(word, num_doc_wrd_usages)) 
AS (word,num_doc_wrd_usages),
    SUM(doc_word_counts.num_doc_wrd_usages) 
AS doc_size
                   ;

term_freqs =  
	FOREACH usage_bag_ds 
		GENERATE 
			doc_id AS doc_id, 
			word AS word, 
			((double) num_doc_wrd_usages / (double) doc_size) AS term_freq; 
		; 

/* Remember to replace X with your group number! */
STORE term_freqs 
INTO '/project/citylabs-workshop/group-X/term_freqs.tsv';
```
Execution should take approximately 10 minutes.

### Step 3 - Compute Inverse Document Frequencies (IDF)
Compute inverse document frequences:

- Input: term_freqs.tsv
- Output: term_usages_idf.tsv

Task: run PIG script to produce the output file

*(Remember to replace X with your group number!!!)*
```
/* Step 3 - Compute Inverse Document Frequency (IDF) */
set mapreduce.map.memory.mb    2048 
set mapreduce.reduce.memory.mb 5120 
term_freqs = LOAD '/project/citylabs-workshop/group-X/term_freqs.tsv'
 	USING PigStorage('\t')
	AS (doc_id:chararray, term:chararray, term_freq:double); 

term_usage_bag  = GROUP term_freqs BY term; 
term_usages    = FOREACH term_usage_bag 
                            GENERATE 
                                FLATTEN(term_freqs) AS (doc_id, term, term_freq), 
                                COUNT(term_freqs)  AS num_docs_with_term
                  ; 
term_usages_idf = FOREACH term_usages { 
              idf    = LOG((double) 48790 / (double) num_docs_with_term);  
                GENERATE 
                  doc_id AS doc_id, 
                  term  AS term, 
                  term_freq as term_freq,
                  idf AS idf 
                ; 
            }; 
/* (Remember to replace X with your group number!!!) */
STORE term_usages_idf INTO '/project/citylabs-workshop/group-X/term_usages_idf.tsv'; 
```
Execution should take approximately 5 minutes.

Task: (HIVE) Inspect term frequency and inverse document frequency of Books ID: 1, 1234, 5307, 19116

```
-- 3.1 - TABLE TERM_USAGES_IDF   
create table citylabs_workshop.terms_usages_idf_X
(DOC STRING, TERM STRING, TF DOUBLE, IDF DOUBLE) 
ROW FORMAT DELIMITED  
FIELDS TERMINATED BY  '\t' 
LINES TERMINATED BY '\n' 
STORED AS TEXTFILE 
LOCATION '/project/citylabs-workshop/group-X/term_usages_idf.tsv'; 

-- 3.2 - TABLE TERM_USAGES_IDF   
-- Id can be any Gutenberg doc_id. Try 5307, 19116, 1, 1234
SELECT DOC, TERM, TF, IDF FROM citylabs_workshop.terms_usages_idf_X
WHERE DOC = 'Gutenberg-1234' order by TF desc;
```

### Step 4 - Compute TF/IDF
Compute TF/IDF

- Input: term_usages_idf.tsv includes TF and IDF
- Output: tfidf.tsv
- Task: run PIG script to produce the output file.

***Remember to replace X with your group number!!!***
```
/* Step 4: Compute TF/IDF */
set mapreduce.map.memory.mb    2048 
set mapreduce.reduce.memory.mb 5120 

word_usages_idf = LOAD '/project/citylabs-workshop/group-X/term_usages_idf.tsv' 
USING PigStorage('\t') AS (doc_id:chararray, word:chararray, tf:double, idf:double); 

tfidf = FOREACH word_usages_idf { 
              tf_idf = (double) tf*idf; 
                GENERATE 
                  doc_id AS doc_id, 
                  word  AS word, 
                  tf_idf AS tf_idf 
                ; 
            }; 
/* (Remember to replace X with your group number!!!) */
STORE tfidf INTO '/project/citylabs-workshop/group-X/tfidf.tsv'; 
```

Task: Compare TF with TFIDF
```
-- 4.1 - TABLE TFIDF 
CREATE TABLE citylabs_workshop.tfidf_X
(DOC STRING, TERM STRING, TFIDF DOUBLE) 
ROW FORMAT DELIMITED  
FIELDS TERMINATED BY  '\t' 
LINES TERMINATED BY '\n' 
STORED AS TEXTFILE 
LOCATION '/project/citylabs-workshop/group-X/tfidf.tsv'; 

-- 4.2 - TABLE TFIDF   
-- Id can be any Gutenberg doc_id. Try 5307, 19116, 1, 1234
SELECT DOC, TERM, TFIDF FROM citylabs_workshop.tfidf_X
WHERE DOC = 'Gutenberg-1234' order by TFIDF desc; 
```


