---
layout: post
title: "Similarity between two datasets"
date: 2016-04-07 06:46:58 +0530
comments: true
categories:  [ C, Linux ]
---
I am writing this post to demonstrate how to find the similarity between two datasets using C programming. Also to see how the efficiency of the application is increased when it is handled in algorithmic approach.

## Scenario
Consider two datasets A and B each having ID, first name and last name. First dataset has 100 K rows and second has 10 million rows. Each of 100 K should scan through 10 million rows and find possible matches. Output dataset should provide a clear idea as how 100 K records in Dataset A matches with one or many records in Dataset B.

## Motivation
Based on the details of the datasets , it seems the first dataset is relatively smaller than the other in the ratio of 1:100 ( ie 100K vs 10 million records ). So I am considering the design pattern of loading the smaller dataset into memory and read the second dataset record one by one and find the match and perform the join.

## Algorithm
``` bash
// Algorithm:
//
//      load dataset1;
//      sort dataset1;                   // using qsort algorithm
//
//      loop
//          read record from dataset2
//          if record match in dataset1 //using binary search algorithm
//          then
//              add record to matchrec in dataset1
//          endif
//      end loop
//
//      display matchrecords
//
```
The efficiency in sorting  ( the first dataset  dataset1) is accomplished using  "qsort" algorithm. The main core logic of this program is how quickly , it can find the match in the first dataset. For this, I made use of “binary search algorithm” . Here is the complexity details of the algorithms used.

> Efficiency of sorting    : qsort    - O(n log n)
> Efficiency of searching  : bsearch  - O(log n)

## Input/Output Format
The dataset record contains identifier, firstname & lastname seperated by “\t” tab space as below,
### Data set Input Format
Both dataset1(smalldataset.txt) & dataset2 (largedataset.txt)

``` bash
/Users/user/wkarea//data $ head smalldataset.txt
12001 	 Aaron2001 	 Aaberg2001
22001 	 Aaron2001 	 Aaby2001
32001 	 Abbey2001 	 Aadland2001
42001 	 Abbie2001 	 Aagaard2001
52001 	 Abby2001 	 Aakre2001
62001 	 Abdul2001 	 Aaland2001
```

### Output Format:

``` bash
{ dataset1 record1 } => {
                                          {dataset2 match record1},
                                          {dataset2 match record2},
                                           .....
                                          {dataset2 match recordN}
                                      }
{ dataset1 record2 } => {
                                          {dataset2 match record1},
                                          {dataset2 match record2},
                                           .....
                                          {dataset2 match recordN}
                                      }
```

## Generating larger dataset
For testing the performance/efficiency of the implementation, I prepared the  datasets with 100K records in dataset1 and 10 million records in dataset2 using the below script which take small dataset and add suffix 0...N so that the names will be unique except few cases where I am gonna insert match records to check the output of the application.

``` bash
~/wkarea//data $ cat gen_bulkdata.sh
#!/bin/bash
cnt=0
>largedataset.txt
while [ $cnt -le 2000 ]
do
  cat dataset.final | awk  \
       '{print $1"'"$cnt"'","\t",$2"'"$cnt"'","\t",$3"'"$cnt"'"}'  >> largedataset.txt
  echo "Counting $cnt"
  let cnt=cnt+1
done
```

## Execution & Testing
For performance testing I mocked dataset in such a way that the records are unique in dataset1 ( 1,00,000 records ) and few match records in dataset2 ( which contains 10 million records in total)

``` bash
~/wkarea//data $ ls -rlth
total 665440
-rw-r--r--  1 user  staff   207B Mar 21 02:33 gen_bulkdata.sh
-rw-r--r--  1 user  staff   322M Mar 21 02:53 largedataset.txt
-rw-r--r--  1 user  staff   3.4M Mar 21 02:54 smalldataset.txt

~/wkarea//data $ wc -l smalldataset.txt largedataset.txt
  100000 smalldataset.txt
 10000000 largedataset.txt
 10100000 total
```

## Output Matched Records
The entire datasets datset1 & dataset2 ( 100K vs 10 million records ) are scanned and completed in 2 mins 8 secs.

``` bash
/Users/user/wkarea//stats $ cat output_withduration
{ 49862200, Sunil, Kumar } =>  {
	{ 234563, Sunil, Kumar },
	{ 986966, Sunil, Kumar },
	{ 49962002, Sunil, Kumar },
    }
{ 49872200, RameshKumar, Bhatia } =>  {
	{ 49902203, RameshKumar, Bhatia },
	{ 21234543, RameshKumar, Bhatia },
    }
      128.55 real       127.91 user         0.52 sys
```

The application is run in the background,

``` bash
$ nohup time ./similar > stats/output_withduration &
[2]	2227

/Users/user/wkarea//stats $ jobs
[2] +  Running                 nohup time ./similar > stats/output_withduration &
[1] -  Running                 nohup top -l 1440 -s 5 -pid 2313 > top_statistics.txt &
```

CPU Usage: ( around 30% spikes in CPU utilization )

## Source Code:

The source code is organized in such a way that it can be used as an API in any other application copying similar.c and similar.h.

``` bash
/Users/user/wkarea/ $ tree
.
├── Makefile
├── data
│   ├── largedataset.txt
│   └── smalldataset.txt
├── inc
│   └── similar.h
├── similar
└── src
    ├── main.c
    └── similar.c

3 directories, 7 files
/Users/user/wkarea/ $
```

### API functions:
* load_dataset1   - Load the dataset 1 into memory.
Usage: void load_dataset1(char *filename)
* join_datasets    - Join dataset1 with dataset2.
Usage: void join_datasets (char *filename);
* print_similar    - Print the similar records in datasets against dataset1.
Usage: void print_similar(void)

src/main.c

``` c
//
//  main.c
//  
//
//  Created by Marikannan Muthiah on 20/03/16.
//
//

#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#include "similar.h"

struct          DATASET DS1[MAX_REC];
long long       DS1_TOTALREC;
struct          MATCHREC  DS2REC;
char            recbuf[RECORDSZ+1];
char            sep[2];
int             jsonfmt;

// Algorithm:
//
//      load dataset1;
//      sort dataset1;                      // using qsort
//
//      loop
//          read record from dataset2
//          if record match in dataset1     // match is found based on binary search
//          then
//              add record to matchrec in dataset1
//          endif
//      end loop
//
//      display matchrecords
//
// Algorithms Used : For better efficiency sorting is performed by "qsort" algorithm and searching the match in dataset1 is accoplished using "binary search".
// Effiency of sorting   : qsort    - O(n log n)
// Effiency of searching : bsearch  - O(log n)


int main(void)
{
	char ds1_fname[FILESZ+1],ds2_fname[FILESZ+1];	/* filename of dataset1 & 2 */
    int  idx = 0, idx2 = 0;
    sep[0]=',';                                     /* defaule seperator used for display */
    jsonfmt = JSONFMT;

	sprintf ( ds1_fname, "./data/smalldataset.txt");
	sprintf ( ds2_fname, "./data/largedataset.txt");


	/* Load the small dataset1 into memory and will be used for join later against dataset 2 */
    load_dataset1(ds1_fname);

    /* sort the dataset 1 */
	qsort(DS1,DS1_TOTALREC,sizeof(struct DATASET),compare_ds1compkey);

    /* find the match for each record in dataset1 in dataset2 */
    join_datasets(ds2_fname);

    /* diplay the join results */
    print_similar();

	return 0;
}
```

src/similar.c

``` c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#include "similar.h"


/* Function : gen_compkey - generate compositekey */
static void gen_compkey(char *compkey, const struct DATASET *ds1) {
	memset(compkey,0x00,COMPKEYSZ);
	snprintf(compkey,COMPKEYSZ,"%s|%s",ds1->firstname,ds1->lastname);
}

/* Function : init_fields - initialize the fileds  */
static void init_fields(struct DATASET *ds1) {
    int idx = 0;

    ds1->id = 0;
    memset(ds1->firstname,0x00,NAMESZ+1);
    memset(ds1->lastname,0x00,NAMESZ+1);
    memset(ds1->compkey,0x00,COMPKEYSZ+1);
    ds1->matchcnt = 0;

    for ( idx = 0 ; idx < MAX_MATCHREC; idx++ ) {
        ds1->matchrec[idx].id = 0;
        memset(ds1->matchrec[idx].firstname,0x00,NAMESZ+1);
        memset(ds1->matchrec[idx].lastname,0x00,NAMESZ+1);
        memset(ds1->matchrec[idx].compkey,0x00,COMPKEYSZ+1);
    }
}

/* Function : get_fields - seperate the fields from the tab '\t' seperated record */
static void get_fields(char *recbuf, struct DATASET *ds1) {
	sscanf(recbuf,"%d%s%s",&ds1->id,ds1->firstname,ds1->lastname);
    memset(ds1->compkey,0x00,COMPKEYSZ);
    snprintf(ds1->compkey,COMPKEYSZ,"%s|%s",ds1->firstname,ds1->lastname);
}

/* Function : get_fields - seperate the fields from the tab '\t' seperated record */
static void get_fields2(char *recbuf, struct MATCHREC *matchrec ) {
    matchrec->id = 0;
    memset(matchrec->firstname,0x00,NAMESZ+1);
    memset(matchrec->lastname,0x00,NAMESZ+1);
	sscanf(recbuf,"%d%s%s",&matchrec->id,matchrec->firstname,matchrec->lastname);
    memset(matchrec->compkey,0x00,COMPKEYSZ);
	snprintf(matchrec->compkey,COMPKEYSZ,"%s|%s",matchrec->firstname,matchrec->lastname);

}

/* Function : print_dsrec - to print a data set record */
static void print_dsrec(const struct DATASET ds,char sep[]) {
    if ( jsonfmt == JSONFMT )
        printf("{ %d, %s, %s } => ",ds.id,ds.firstname,ds.lastname);
    else
        printf("%d%s%s%s%s \n",ds.id,sep,ds.firstname,sep,ds.lastname);
}

/* Function : print_matchrec - to print a data set 2 record preceding with single tab */
static void print_matchrec(const struct MATCHREC matchrec ,char sep[]) {
    if ( jsonfmt == JSONFMT )
        printf("\t{ %d, %s, %s }",matchrec.id,matchrec.firstname,matchrec.lastname);
    else
        printf("\t%d%s%s%s%s \n",matchrec.id,sep,matchrec.firstname,sep,matchrec.lastname);
}

/* Function : print_ds - to print the entire data set */
static void print_ds(void) {
	int idx = 0;

	for ( idx = 0; idx < DS1_TOTALREC; idx++ )
		print_dsrec(DS1[idx],sep);

	printf("Total number of records : %lld\n",DS1_TOTALREC);

}

/* Function : add_matchrec - To add match record */
static void add_matchrec (struct DATASET *ds1rec, struct MATCHREC  *matchrec ) {
    if ( ds1rec->matchcnt+1 <= MAX_MATCHREC ) {
        ds1rec->matchrec[ds1rec->matchcnt].id = matchrec->id;
        memcpy(ds1rec->matchrec[ds1rec->matchcnt].firstname,matchrec->firstname,NAMESZ);
        memcpy(ds1rec->matchrec[ds1rec->matchcnt].lastname,matchrec->lastname,NAMESZ);
        memcpy(ds1rec->matchrec[ds1rec->matchcnt].compkey,matchrec->compkey,COMPKEYSZ);
        ds1rec->matchcnt += 1;
    }
}

/* Function : compare_ds1compkey - comparator function for qsort algorithm */
int compare_ds1compkey (struct DATASET *ds1rec1, struct DATASET *ds1rec2) {
	char compkey1[COMPKEYSZ];
	char compkey2[COMPKEYSZ];

	memset(compkey1,0x00,COMPKEYSZ);
	memset(compkey2,0x00,COMPKEYSZ);

	gen_compkey(compkey1,ds1rec1);
	gen_compkey(compkey2,ds1rec2);

	return(memcmp(compkey1,compkey2,COMPKEYSZ));
}

/* Function : bsearch_compare - comparator function for binary search algorithm  */
int bsearch_compare(char *recbuf, struct DATASET *ds1rec) {
    char compkey1[COMPKEYSZ];
	char compkey2[COMPKEYSZ];
    struct MATCHREC matchrec ;
    int rc = 0;
    int idx = 0;

	memset(compkey2,0x00,COMPKEYSZ);
    get_fields2(recbuf,&matchrec);
	gen_compkey(compkey2,ds1rec);

    rc = memcmp(matchrec.compkey,compkey2,COMPKEYSZ);
    if ( 0 == rc )
        add_matchrec(ds1rec,&matchrec);

	return(rc);
}

/* Function: load_dataset1 - Load small dataset which can handle upto 1,00,000 records */
void load_dataset1(char *ds1_fname ) {
    FILE *fpds1;
	int  idx = 0;

	fpds1 = fopen(ds1_fname, "r" );
	if ( fpds1 == NULL ) {
		printf ("Error : Not able to open data set 1 : %s. \nExitting ...\n",ds1_fname);
		exit (-1);
	}
    memset(recbuf,0x00,RECORDSZ);               /* clear record buffer */
    memset(recbuf,0x00,256);               /* clear record buffer */

	idx = 0;
	while ( fgets (recbuf,256,fpds1) != NULL ) {
        init_fields(&DS1[idx]);
		get_fields(recbuf,&DS1[idx]);
		// print_dsrec(DS1[idx],"|");		/* '|' pipe is a seperator while displaying */
		idx++;
	}
	DS1_TOTALREC = idx ; 	/* total number of records loaded into memory. */
}

/* Function : join_datasets - Function which joins two datasets dataset1 & dataset2 */
void join_datasets (char *ds2_fname) {
    int  idx = 0;
    FILE *fpds2; 				/* file pointer to data set1 & data set 2 */

    fpds2 = fopen(ds2_fname, "r" );
	if ( fpds2 == NULL ) {
		printf ("Error : Not able to open data set 2 : %s. \nExitting ...\n",ds2_fname);
		exit (-1);
	}
    memset(recbuf,0x00,RECORDSZ);               /* clear record buffer */

	idx = 0;
	while ( fgets (recbuf,256,fpds2) != NULL ) {
		get_fields2(recbuf,&DS2REC);
        bsearch(recbuf,DS1,DS1_TOTALREC,sizeof(struct DATASET),&bsearch_compare);
		idx++;
	}
}

/* Function : print_similar - To print similar records b/w dataset1 and dataset 2 */
void print_similar(void) {
    int  idx = 0, idx2 = 0;

    /* diplay the join results */
    idx=0;
    for ( idx =0; idx < DS1_TOTALREC ; idx++ ) {
        if ( DS1[idx].matchcnt ) {
            print_dsrec(DS1[idx],sep);
            if ( jsonfmt == JSONFMT )
                printf(" { \n");

            for ( idx2 = 0 ; idx2 < MAX_MATCHREC && DS1[idx].matchrec[idx2].id != 0; idx2++ ) {
                print_matchrec(DS1[idx].matchrec[idx2],sep);
                jsonfmt == JSONFMT ? printf(",\n") : printf("\n") ;
            }
            if ( jsonfmt == JSONFMT )
                printf("    } \n");
        }
    }
}
```

inc/similar.h

``` c
#define NAMESZ 		  20			/* first , last name size */
#define FILESZ 		  256			/* file name size */
#define COMPKEYSZ	  40			/* composite key size first name & last name */
#define RECORDSZ	  256			/* dataset record size */
#define MAX_REC		  100000 		/* maximum number of records in dataset 1 */
#define MAX_MATCHREC  	  5             /* maximum number of match records */


struct MATCHREC {
    int id;
    char firstname[NAMESZ+1];
    char lastname[NAMESZ+1];
    char compkey[COMPKEYSZ+1];
};

struct DATASET {
	int id;
	char firstname[NAMESZ+1];
	char lastname[NAMESZ+1];
	char compkey[COMPKEYSZ+1];
    int matchcnt;
    struct MATCHREC matchrec[MAX_MATCHREC];
};

extern struct DATASET   DS1[MAX_REC];
extern struct MATCHREC   DS2REC;
extern long long        DS1_TOTALREC;
extern char             recbuf[RECORDSZ+1];
extern char            sep[2];
extern int             jsonfmt;
#define JSONFMT     1
#define NOT_JSONFMT  0


/* Function : compare_ds1compkey - comparator function for qsort algorithm */
extern int compare_ds1compkey (struct DATASET *ds1rec1, struct DATASET *ds1rec2);
/* Function : bsearch_compare - comparator used by binary search algorithm */
extern int bsearch_compare(char *recbuf, struct DATASET *ds1rec);
/* Function: load_dataset1 - Load small dataset which can handle upto 1,00,000 records */
extern void load_dataset1(char *ds1_fname );
/* Function : join_datasets - Function which joins two datasets dataset1 & dataset2 */
extern void join_datasets (char *ds2_fname);
/* Function : print_similar - To print similar records b/w dataset1 and dataset 2 */
extern void print_similar(void);
```
