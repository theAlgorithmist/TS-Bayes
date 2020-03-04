# Typescript Math Toolkit Naive Bayes

Naive Bayes is one of those classifiers that I thought would never survive a first-semester college stats course.  Well, that's what I get for thinking :)

Although the technique is useful under the right circumstances, it does require a particularly experienced/skilled person to determine proper use cases.  I'll leave that decision to others and have in the mean time implemented Naive Bayes (with a comprehensive frequency table class) in the Typescript Math Toolkit.  


Author:  Jim Armstrong - [The Algorithmist]

@algorithmist

theAlgorithmist [at] gmail [dot] com

Typescript: 2.0.3

Version: 1.0


## Installation

Installation involves all the usual suspects

  - npm and gulp installed globally
  - Clone the repository
  - npm install
  - get coffee (this is the most important step)


### Building and running the tests

1. gulp compile

2. gulp test

The test suite is in Mocha/Chai and specs reside in the _test_ folder.



### Introduction

The _Typescript Math Toolkit_ implementation of Naive Bayes is tied to a frequency table.  This allows raw data to be entered all at one time or cell frequencies may be updated individually as part of an interactive experiment.  

The frequency table may be validated, and some remedial action may be applied to help compensate for incorrect data.  Laplace estimation will likely be provided in a future update.

The current release is provided to obtain API feedback and encourage additional testing on practical problems.


### Contents

This first release of Naive Bayes in the _Typescript Math Toolkit_ consists of two classes, _TSMT$FrequencyTable_ and _TSMT$Bayes_.  The former may be used independently or as part of naive Bayesian analysis.  Its public API is listed below.


```
public clone(): TSMT$FrequencyTable
public setCellFrequency(row: number, col: number, value: number): void
public setupTable(rowLabels: Array<string>, colLabels: Array<string>): void
public fromArray( data: Array< Array<number> >, rowLabels: Array<string>, colLabels: Array<string>, rowTotals: Array<number> ): void
public get columnLabels(): Array<string>
public get rowLabels(): Array<string>
public get rowCount(): number
public get colCount(): number
public get rowTotals(): Array<number>
public get columnTotals(): Array<number>
public get classSum(): number
public get table(): Array< Array<number> >
public getColumn(label:string): Array<number>
public getClassProb(className: string): number
public getPrior(predictor: string): number
public getConditional(predictor: string, className: string): number
public removeColumn(category: string): void
public addCellFrequency(rowLabel: string, colLabel: string, count: number=1): void
public addRowCount(rowLabel: string, count: number=1): void
public validate(round: boolean=false): boolean
public columnCorrelation(column1Label: string, column2Label: string): number
```

The public API for the _TSMT$Bayes_ class is

```
public set table(freqTable: TSMT$FrequencyTable)
public naive(className: string, predictors: Array<string>, numeratorOnly: boolean = false, avoidUnderflow: boolean = false): number
public clear(): void
```


### Usage

A frequency table may be initialized at one time, or the table structure may be setup and then frequencies are added individually as data becomes available.  The former approach is illustrated below.


```
const spamTestTable : TSMT$FrequencyTable = new TSMT$FrequencyTable();

const data: Array<Array<number>> = new Array< Array<number> >();
const rows: Array<string>        = [ "Spam", "Not Spam" ];
const cols: Array<string>        = [ "Viagra Y", "Viagra N", "Money Y", "Money N", "Groceries Y", "Groceries N", "Unsubscribe Y", "Unsubscribe N" ];
const rowTotals: Array<number>   = [20, 80];

data.push( [4, 16, 10, 10, 0, 20, 12, 8] );
data.push( [1, 79, 14, 66, 8, 71, 23, 57] );

spamTestTable.fromArray( data, rows, cols, rowTotals );

```

The latter use case for the same data is illustrated in the following code segment


```
const rows: Array<string> = [ "Spam", "Not Spam" ];
const cols: Array<string> = [ "Viagra Y", "Viagra N", "Money Y", "Money N", "Groceries Y", "Groceries N", "Unsubscribe Y", "Unsubscribe N" ];
.
.
.
table.setupTable(rows, cols);
 
table.addCellFrequency( "Spam", "Viagra Y"      , 4  );
table.addCellFrequency( "Spam", "Viagra N"      , 16 );
table.addCellFrequency( "Spam", "Money Y"       , 10 );
table.addCellFrequency( "Spam", "Money N"       , 10 );
table.addCellFrequency( "Spam", "Groceries N"   , 0  );
table.addCellFrequency( "Spam", "Unsubscribe Y" , 12 );
table.addCellFrequency( "Spam", "Unsubscribe N" , 8  );

table.addCellFrequency( "Not Spam", "Viagra Y"      , 1  );
table.addCellFrequency( "Not Spam", "Viagra N"      , 79 );
table.addCellFrequency( "Not Spam", "Money Y"       , 14 );
table.addCellFrequency( "Not Spam", "Money N"       , 66 );
table.addCellFrequency( "Not Spam", "Groceries Y"   , 8  );
table.addCellFrequency( "Not Spam", "Groceries N"   , 71 );
table.addCellFrequency( "Not Spam", "Unsubscribe Y" , 23 );
table.addCellFrequency( "Not Spam", "Unsubscribe N" , 57 );

table.addRowCount( "Spam"    , 20 );
table.addRowCount( "Not Spam", 80 );
```

Table columns may be analyzed for a high degree of correlation, after which one of the columns may be removed.  The table frequency data may be queried and all such queries preserve immutability of internal table data.

The table may be validated and attempts may be made during the validation process to compensate for data issues.  The relevant call is

```
const valid: boolean = table.validate(true);
.
.
or
.
.
const valid: boolean = table.validate();
```

The input argument controls whether or not rounding to nearest integer is applied to cell data.  The _validate()_ method also checks for

* Any row total that is not a number or is less than zero (no internal action)

* Any cell value that is not a number or is less than zero (sets the cell frequency to zero)

* Any cell value that is greater than the row total for that cell's row (clips to row total)

Any occurrence of these conditions generates a false return and is an indication that examination of the frequency table inputs is indicated.  It may still be possible to perform an analysis based on the frequency table, but such action is not recommended.

The auto-correction feature is experimental and is subject to future expansion or removal, based on user feedback.

After definition and analysis of the frequency table is complete, a naive Bayes analysis is performed by assigning the table to a _TSMT$Bayes_ instance. Then, select a class and list of predictors.

In many cases, the same list of predictors is run across multiple classes, in which case the denominator of the Bayes equation is the same.  It is possible to request that the numerator only be returned instead of the complete computation.

Two sample use cases are provided below.  It is presumed that a frequency table has already been defined and initialized with data.

```
const bayes: TSMT$Bayes          = new TSMT$Bayes();
const table: TSMT$FrequencyTable = new TSMT$FrequencyTable();
.
.
.
    
bayes.table = table;
   
// probability golf will be played given that the day is sunny
const result: number = bayes.naive( "Golf Y", ["Sunny"] );
```

```
// example is from 'Machine Learning in R' by Lantz
const bayes: TSMT$Bayes = new TSMT$Bayes();

// set the frequency table
bayes.table = spamTestTable;

// probability of spam given 'Viagra Y', 'Money N', 'Groceries N', and 'Unsubscribe Y', i.e. a message contains the
// words 'Viagra' and 'Unsubscribe' - numerator only
const pSpam: number = bayes.naive('Spam', ['Viagra Y', 'Money N', 'Groceries N', 'Unsubscribe Y'], true);

// probability of not spam given the same word set
const pNotSpam: number = bayes.naive('Not Spam', ['Viagra Y', 'Money N', 'Groceries N', 'Unsubscribe Y'], true);

// total likelihood of spam or not spam
const total: number = pSpam + pNotSpam;

// probability of spam
const probSpam: number = pSpam/total;

// probability of not spam
const probNotSpam: number = pNotSpam/total;
```

Refer to the specs in the _test_ folder for more usage examples.


### Notes

Internally, the frequency tables stores two-dimensional data column-major.  Remember this when processing information from the _table_ accessor.

This is an alpha release and the API is subject to future change.  Likely changes include the ability to apply a Laplace estimator and provide an option for information gain metrics.

In very limited cases, it is possible for numerical issues (even underflow) to occur with large models that have a number of predictors with very low frequency of occurrence.  It is possible to avoid numerical issues via use of the _avoidUnderflow_ flag in the _TSMT$Bayes naive()_ method.                                                                                                       

License
----

Apache 2.0

**Free Software? Yeah, Homey plays that**

[//]: # (kudos http://stackoverflow.com/questions/4823468/store-comments-in-markdown-syntax)

[The Algorithmist]: <https://www.linkedin.com/in/jimarmstrong>

