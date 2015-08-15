# Introduction #

DiskDoubler's algorithms numbers 2 and 5 appears to use an extremely uncommon compression method, splay tree compression. A splay tree is used like a Huffman tree to perform adaptive prefix coding. More about splay tree compression can be found at http://www.fadden.com/techmisc/hdc/lesson05.htm.

DiskDoubler uses 256 splay trees (or optionally less, for algorithm 5) to encode individual bytes. Which tree to use for each byte is determined by the previous byte (if less than 256 trees are used, the low-order bits are used to select the tree).

See http://code.google.com/p/theunarchiver/source/browse/XADMaster/XADDiskDoublerMethod2Handle.m for the specifics of the algorithm, or see the stand-alone implementation provided below.

# Benchmarks #

## The Canterbury Corpus ##

|   | text | fax | Csrc | Excl | SPRC | tech | poem | html | list | man | play | Average |
|:--|:-----|:----|:-----|:-----|:-----|:-----|:-----|:-----|:-----|:----|:-----|:--------|
| DD2 | 4.03 | 1.58 | 3.54 | 2.99 | 3.71 | 4.03 | 4.04 | 4.33 | 3.67 | 4.39 | 3.97 | 3.66    |
| gzip-b | **2.85** | **0.82** | **2.24** | **1.63** | **2.67** | **2.71** | **3.23** | **2.59** | **2.65** | **3.31** | **3.12** | **2.53** |
| gzip-f | 3.43 | 1.02 | 2.63 | 1.90 | 2.96 | 3.26 | 3.80 | 2.94 | 2.89 | 3.53 | 3.63 | 2.91    |
| compress | 3.27 | 0.97 | 3.56 | 2.41 | 4.21 | 3.06 | 3.38 | 3.68 | 3.90 | 4.43 | 3.51 | 3.31    |
| lzrw1 | 4.94 | 2.05 | 3.61 | 2.53 | 4.55 | 4.79 | 5.44 | 4.26 | 4.02 | 4.68 | 5.16 | 4.18    |
| char | 4.59 | 1.22 | 5.14 | 3.58 | 5.38 | 4.68 | 4.54 | 5.31 | 4.96 | 5.20 | 4.83 | 4.49    |

## The Artificial Corpus ##

|   | a | aaa | alphabet | random | Average |
|:--|:--|:----|:---------|:-------|:--------|
| DD2 | **8.00** | 1.00 | 1.00     | 7.31   | **4.33** |
| gzip-b | 168.00 | **0.01** | **0.02** | **6.05** | 43.52   |
| gzip-f | 168.00 | 0.04 | 0.05     | 6.18   | 43.57   |
| compress | 40.00 | 0.04 | 0.24     | 7.39   | 11.92   |
| lzrw1 | 104.00 | 1.07 | 1.07     | 8.00   | 28.53   |
| char | 128.00 | 0.03 | 4.74     | **6.05** | 34.71   |

## The Large Corpus ##

|   | E.coli | bible | world | Average |
|:--|:-------|:------|:------|:--------|
| DD2 | 2.40   | 3.73  | 4.11  | 3.41    |
| gzip-b | 2.24   | **2.33** | **2.33** | **2.30** |
| gzip-f | 2.63   | 2.95  | 2.97  | 2.85    |
| compress | 2.17   | 2.77  | 3.19  | 2.71    |
| lzrw1 | 5.02   | 4.36  | 5.03  | 4.80    |
| char | **2.01** | 4.35  | 5.00  | 3.79    |

## The Miscellaneous Corpus ##

|   | pi | Average |
|:--|:---|:--------|
| DD2 | 3.98 | 3.98    |
| gzip-b | 3.76 | 3.76    |
| gzip-f | 3.98 | 3.98    |
| compress | 3.75 | 3.75    |
| lzrw1 | 5.81 | 5.81    |
| char | **3.33** | **3.33** |

## The Calgary Corpus ##

|   | bib | book1 | book2 | geo | news | obj1 | obj2 | paper1 | paper2 | pic | progc | progl | progp | trans | Average |
|:--|:----|:------|:------|:----|:-----|:-----|:-----|:-------|:-------|:----|:------|:------|:------|:------|:--------|
| DD2 | 4.11 | 4.23  | 4.14  | **5.18** | 4.49 | 4.64 | 3.66 | 4.17   | 4.15   | 1.58 | 3.96  | 3.36  | 3.38  | 3.62  | 3.90    |
| gzip-b | **2.51** | **3.25** | **2.70** | 5.34 | **3.06** | **3.84** | **2.63** | **2.79** | **2.89** | **0.82** | **2.68** | **1.80** | **1.81** | **1.61** | **2.69** |
| gzip-f | 3.15 | 3.80  | 3.26  | 5.45 | 3.48 | 3.98 | 3.04 | 3.25   | 3.41   | 1.02 | 3.12  | 2.24  | 2.17  | 2.05  | 3.10    |
| compress | 3.35 | 3.46  | 3.28  | 6.08 | 3.86 | 5.23 | 4.17 | 3.77   | 3.52   | 0.97 | 3.87  | 3.03  | 3.11  | 3.27  | 3.64    |
| lzrw1 | 4.77 | 5.47  | 4.75  | 6.77 | 4.94 | 4.93 | 4.13 | 4.63   | 4.90   | 2.05 | 4.37  | 3.53  | 3.43  | 3.71  | 4.46    |
| char | 5.23 | 4.54  | 4.80  | 5.66 | 5.20 | 6.01 | 6.28 | 5.02   | 4.63   | 1.22 | 5.25  | 4.80  | 4.91  | 5.56  | 4.94    |

# Implementation #

If anyone is interested in this unusual and simple algorithm, here are stand-alone implementations of both a compressor and an uncompressor.

## Compressor ##

```
#include <stdint.h>
#include <stdio.h>
#include <stdlib.h>



typedef struct BitWriter
{
	FILE *fh;
	int bits,numbits;
} BitWriter;

void InitBitWriter(BitWriter *self,FILE *fh)
{
	self->fh=fh;
	self->bits=0;
	self->numbits=0;
}

void WriteBit(BitWriter *self,int bit)
{
	self->bits=(self->bits<<1)|bit;
	self->numbits++;

	if(self->numbits==8)
	{
		fputc(self->bits,self->fh);
		self->bits=0;
		self->numbits=0;
	}
}

void StopBitWriter(BitWriter *self)
{
	if(self->numbits)
	{
		fputc(self->bits<<8-self->numbits,self->fh);
	}
}



typedef struct DDMethod2State
{
	int numtrees,currtree;
	struct
	{
		uint8_t parents[512];
		uint16_t leftchildren[256];
		uint16_t rightchildren[256];
	} trees[256];
} DDMethod2State;

void InitDDMethod2State(DDMethod2State *self,int numtrees)
{
	self->numtrees=numtrees;
	self->currtree=0;

	for(int i=0;i<numtrees;i++)
	{
		for(int j=0;j<256;j++)
		{
			self->trees[i].parents[2*j]=j;
			self->trees[i].parents[2*j+1]=j;
			self->trees[i].leftchildren[j]=j*2;
			self->trees[i].rightchildren[j]=j*2+1;
		}
	}
}

void UpdateDDMethod2State(DDMethod2State *self,int byte)
{
	uint8_t *parents=self->trees[self->currtree].parents;
	uint16_t *leftchildren=self->trees[self->currtree].leftchildren;
	uint16_t *rightchildren=self->trees[self->currtree].rightchildren;

	int node=byte+0x100;
	for(;;)
	{
		int parent=parents[node];
		if(parent==1) break;

		int grandparent=parents[parent];

		int uncle=leftchildren[grandparent];
		if(uncle==parent)
		{
			uncle=rightchildren[grandparent];
			rightchildren[grandparent]=node;
		}
		else
		{
			leftchildren[grandparent]=node;
		}

		if(leftchildren[parent]!=node) rightchildren[parent]=uncle;
		else leftchildren[parent]=uncle;

		parents[node]=grandparent;
		parents[uncle]=parent;

		node=grandparent;
		if(node==1) break;
	}

	self->currtree=byte%self->numtrees;
}

static void RecursivePack(DDMethod2State *self,BitWriter *writer,int node)
{
	if(node==1) return;

	int parent=self->trees[self->currtree].parents[node];

	RecursivePack(self,writer,parent);

	if(self->trees[self->currtree].leftchildren[parent]==node) WriteBit(writer,0);
	else WriteBit(writer,1);
}

void PackDDMethod2Byte(DDMethod2State *self,BitWriter *writer,int byte)
{
	RecursivePack(self,writer,byte+0x100);
	UpdateDDMethod2State(self,byte);
}





int main(int argc,const char **argv)
{
	if(argc!=2)
	{
		fprintf(stderr,"Usage: %s trees <infile >outfile\n",argv[0]);
		return 1;
	}

	BitWriter writer;
	InitBitWriter(&writer,stdout);

	DDMethod2State state;
	InitDDMethod2State(&state,atoi(argv[1]));

	for(;;)
	{
		int byte=fgetc(stdin);
		if(byte==EOF) break;

		PackDDMethod2Byte(&state,&writer,byte/*^0x5a*/);
	}

	StopBitWriter(&writer);

	return 0;
}
```

## Uncompressor ##

```
#include <stdint.h>
#include <stdio.h>
#include <stdlib.h>



typedef struct BitReader
{
	FILE *fh;
	int bits,numbits;
} BitReader;

void InitBitReader(BitReader *self,FILE *fh);
int ReadBit(BitReader *self);

void InitBitReader(BitReader *self,FILE *fh)
{
	self->fh=fh;
	self->bits=0;
	self->numbits=0;
}

int ReadBit(BitReader *self)
{
	if(self->numbits==0)
	{
		self->bits=fgetc(self->fh);
		self->numbits=8;
	}

	self->numbits--;
	return (self->bits>>self->numbits)&1;
}



typedef struct DDMethod2State
{
	int numtrees,currtree;
	struct
	{
		uint8_t parents[512];
		uint16_t leftchildren[256];
		uint16_t rightchildren[256];
	} trees[256];
} DDMethod2State;

void InitDDMethod2State(DDMethod2State *self,int numtrees)
{
	self->numtrees=numtrees;
	self->currtree=0;

	for(int i=0;i<numtrees;i++)
	{
		for(int j=0;j<256;j++)
		{
			self->trees[i].parents[2*j]=j;
			self->trees[i].parents[2*j+1]=j;
			self->trees[i].leftchildren[j]=j*2;
			self->trees[i].rightchildren[j]=j*2+1;
		}
	}
}

void UpdateDDMethod2State(DDMethod2State *self,int byte)
{
	uint8_t *parents=self->trees[self->currtree].parents;
	uint16_t *leftchildren=self->trees[self->currtree].leftchildren;
	uint16_t *rightchildren=self->trees[self->currtree].rightchildren;

	int node=byte+0x100;
	for(;;)
	{
		int parent=parents[node];
		if(parent==1) break;

		int grandparent=parents[parent];

		int uncle=leftchildren[grandparent];
		if(uncle==parent)
		{
			uncle=rightchildren[grandparent];
			rightchildren[grandparent]=node;
		}
		else
		{
			leftchildren[grandparent]=node;
		}

		if(leftchildren[parent]!=node) rightchildren[parent]=uncle;
		else leftchildren[parent]=uncle;

		parents[node]=grandparent;
		parents[uncle]=parent;

		node=grandparent;
		if(node==1) break;
	}

	self->currtree=byte%self->numtrees;
}

int UnpackDDMethod2Byte(DDMethod2State *self,BitReader *reader)
{
	int node=1;
	for(;;)
	{
		int bit=ReadBit(reader);

		if(bit==1) node=self->trees[self->currtree].rightchildren[node];
		else node=self->trees[self->currtree].leftchildren[node];

		if(node>=0x100)
		{
			int byte=node-0x100;

			UpdateDDMethod2State(self,byte);

			return byte;
		}
	}

}



int main(int argc,const char **argv)
{
	if(argc!=3)
	{
		fprintf(stderr,"Usage: %s trees length <infile >outfile\n",argv[0]);
		return 1;
	}

	BitReader reader;
	InitBitReader(&reader,stdin);

	DDMethod2State state;
	InitDDMethod2State(&state,atoi(argv[1]));

	int length=atoi(argv[2]);
	for(int i=0;i<length;i++)
	{
		fputc(UnpackDDMethod2Byte(&state,&reader)/*^0x5a*/,stdout);
	}

	return 0;
}
```