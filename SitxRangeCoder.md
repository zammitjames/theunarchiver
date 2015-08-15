# Overview #

The range coder used in all sitx compressors is functionally similar to Dmitry Subbotin's carryless rangecoder from the PPMd project.

## Initialization ##

```
low=0;
code=0;
range=-1;

for(int i=0;i<4;i++) code=(code<<8)|NextByteFromStream();
```

## Fetching a symbol ##

Here, `freqtable` is an array of symbol frequencies, and `num` is the number of symbols.

```
unsigned int totalfreq=0;
for(int i=0;i<num;i++) totalfreq+=freqtable[i];

range=range/totalfreq;
unsigned int tmp=(code-low)/range;

unsigned int cumulativefreq=0;
unsigned int n=0;
while(n<num-1 && cumulativefreq+freqtable[n]<=tmp)
{
    cumulativefreq+=freqtable[n++];
}

low=low+range*cumulativefreq;
range=range*freqtable[n];

```

The index of the decoded symbol is `n`. This is followed by normalization:

## Normalization ##

```
for(;;)
{
    if( (low^(low+range))>=0x1000000 )
    {
        if(range>=0x10000) break;
        else range=-low&0xffff;
    }
    
    code=(code<<8) | NextByteFromStream();
    range=range<<8;
    low=low<<8;
}
```