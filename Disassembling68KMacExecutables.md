# Introduction #

[IDA](http://www.hex-rays.com/idapro/) can disassemble 68k code just fine, but it does not understand the somewhat complicated format of old Mac OS executables. Here are two IDC scripts that help disassemble System 7 executables.

Here are the steps needed:

  * Get a copy of the resource fork of the executable. This can be done from the command line with something like `cp OldMacApp/..namedfork/rsrc OldMacApp.rsrc`.
  * Copy the .rsrc file to a machine that runs IDA, and open it as a binary file. Set the processor to something suitable, such as 68000 or 68020. Let it load at the default 0 address.
  * Run the first IDC script, `mac_os_resource.idc`. It will parse the resource file structure, and create labels and comments for the various parts of the file. It will also identify all procedures referenced by the jump table used for calling across segments.
  * Run the second IDC script, `mac_os_fixjumps.idc`. This will find all `jsr` instructions that jump through the jump table, and fix their operands up so that the target can be easily identified, and set up proper code xrefs.
  * If you later identify and disassemble more functions, you can re-run `mac_os_fixjumps.idc` to fix the `jsr` instructions in the new parts.
  * Note that `mac_os_fixjumps.idc` will indicate a jump target in the table that is two bytes off. This is to make it easier to identify the real target. The jump table contains an offset followed by a load or jump instruction. A real jump would jump to the instruction following the offset, but the script sets the reference to point at the index instead.

# mac\_os\_resource.idc #

```
#include <idc.idc>

static main()
{
	auto i,j;
	auto resdata,resmap,restypelist,resnamelist;
	auto numtypes,numrefs;
	auto code0,length;

	MakeDword(0);
	OpOff(0,0,0);
	MakeComm(0,"Offset to resource data");

	MakeDword(4);
	OpOff(4,0,0);
	MakeComm(4,"Offset to resource map");

	MakeDword(8);
	OpNumber(8,0);
	MakeComm(8,"Length of resource data");

	MakeDword(12);
	OpNumber(12,0);
	MakeComm(12,"Length of resource map");

	resdata=Dword(0);
	resmap=Dword(4);

	MakeNameEx(resdata,"ResourceData",0);
	MakeNameEx(resmap,"ResourceMap",0);



	MakeDword(resmap);
	OpOff(resmap,0,0);
	MakeComm(resmap,"Offset to resource data");

	MakeDword(resmap+4);
	OpOff(resmap+4,0,0);
	MakeComm(resmap+4,"Offset to resource map");

	MakeDword(resmap+8);
	OpNumber(resmap+8,0);
	MakeComm(resmap+8,"Length of resource data");

	MakeDword(resmap+12);
	OpNumber(resmap+12,0);
	MakeComm(resmap+12,"Length of resource map");

	MakeDword(resmap+16);
	OpNumber(resmap+16,0);
	MakeComm(resmap+16,"Reserved for handle to next resource map");

	MakeWord(resmap+20);
	OpNumber(resmap+20,0);
	MakeComm(resmap+20,"Reserved for file reference number");

	MakeWord(resmap+22);
	OpNumber(resmap+22,0);
	MakeComm(resmap+22,"Resource fork attributes");

	MakeWord(resmap+24);
	OpOffEx(resmap+24,0,REF_OFF32,-1,resmap,-2);
	MakeComm(resmap+24,"Offset to type list");

	MakeWord(resmap+26);
	OpOffEx(resmap+26,0,REF_OFF32,-1,resmap,-2);
	MakeComm(resmap+26,"Offset to name list");

	MakeWord(resmap+28);
	OpNumber(resmap+28,0);
	MakeComm(resmap+28,"Number of types minus one");

	restypelist=resmap+Word(resmap+24)+2;
	resnamelist=resmap+Word(resmap+26)+2;

	MakeNameEx(restypelist,"ResourceTypeList",0);
	MakeNameEx(resnamelist,"ResourceNameList",0);

	numtypes=Word(resmap+28)+1;

	for(i=0;i<numtypes;i++)
	{
		auto entry,reflist,resname;
		entry=restypelist+i*8;

		MakeStr(entry+0,entry+4);
		MakeComm(entry+0,"Resource type");

		MakeWord(entry+4);
		OpNumber(entry+4,0);
		MakeComm(entry+4,"Number of resource of this type minus one");

		MakeWord(entry+6);
		OpOffEx(entry+6,0,REF_OFF32,-1,restypelist,2);
		MakeComm(entry+6,"Offset of reference list for this type");

		reflist=restypelist+Word(entry+6)-2;

		resname=GetString(entry+0,4,ASCSTR_C);
		MakeNameEx(reflist,"ReferenceList"+resname,0);

		numrefs=Word(entry+4)+1;

		for(j=0;j<numrefs;j++)
		{
			auto ref,resid,attrs,data;
			ref=reflist+j*12;

			MakeWord(ref+0);
			OpNumber(ref+0,0);
			MakeComm(ref+0,"Resource ID");

			MakeWord(ref+2);
			OpOffEx(ref+2,0,REF_OFF32,-1,resnamelist,0);
			MakeComm(ref+2,"Offset to resource name");

			MakeDword(ref+4);
			attrs=Byte(ref+4);
			OpOffEx(ref+4,0,REF_OFF32,-1,resdata,attrs<<24);
			MakeComm(ref+4,"Offset to resource data plus attributes");

			MakeDword(ref+8);
			OpNumber(ref+8,0);
			MakeComm(ref+8,"Reserved for handle to resource");

			resid=ltoa(Word(ref+0),10);
			data=resdata+Dword(ref+4)-(attrs<<24);
			MakeNameEx(data,resname+"Resource"+resid,0);

			MakeDword(data);
			OpNumber(data,0);
			MakeComm(data,"Length of resource data");

			if(resname=="CODE" && resid!="0")
			{
				MakeWord(data+4);
				OpNumber(data+4,0);
				MakeComm(data+4,"Offset of first entry in jump table");

				MakeWord(data+6);
				OpNumber(data+6,0);
				MakeComm(data+6,"Number of entries in jump table");
			}
		}
	}

	code0=LocByName("CODEResource0");

	MakeDword(code0+4);
	OpNumber(code0+4,0);
	MakeComm(code0+4,"Size above A5");

	MakeDword(code0+8);
	OpNumber(code0+8,0);
	MakeComm(code0+8,"Size of globals");

	MakeDword(code0+12);
	OpNumber(code0+12,0);
	MakeComm(code0+12,"Length of jump table");

	MakeDword(code0+16);
	OpNumber(code0+16,0);
	MakeComm(code0+16,"A5 offset of jump table");

	MakeNameEx(code0+20,"JumpTable",0);

	length=Dword(code0+12);

	for(i=0;i<length;i=i+8)
	{
		auto jumpentry,jumpresid,resoffs,funcoffs;
		jumpentry=code0+20+i;

		jumpresid=ltoa(Word(jumpentry+4),10);
		resoffs=LocByName("CODEResource"+jumpresid);

		MakeWord(jumpentry+0);
		OpOffEx(jumpentry+0,0,REF_OFF32,-1,resoffs,-8);
		MakeComm(jumpentry+0,"Offset of function");

		MakeWord(jumpentry+2);
		OpNumber(jumpentry+2,0);
		MakeComm(jumpentry+2,"Push instruction");

		MakeWord(jumpentry+4);
		OpNumber(jumpentry+4,0);
		MakeComm(jumpentry+4,"Resource ID to push");

		MakeWord(jumpentry+6);
		OpNumber(jumpentry+6,0);
		MakeComm(jumpentry+6,"LoadSeg instruction");

		funcoffs=resoffs+8+Word(jumpentry+0);
		AutoMark(funcoffs,AU_PROC);
	}
}
```

# mac\_os\_fixjumps.idc #

```
#include <idc.idc>

static main()
{
	auto code0,jumptable,a5offs;
	auto addr;

	code0=LocByName("CODEResource0");
	jumptable=code0+20;
	a5offs=Dword(code0+16);

	addr=0;
	for(;;)
	{
		auto op;
		auto jumpentry,jumpresid,resoffs,funcoffs;

		addr=FindCode(addr,SEARCH_DOWN);
		if(addr==BADADDR) break;

		if(GetMnem(addr)!="jsr") continue;
		if(GetOpType(addr,0)!=4) continue;
		op=GetOpnd(addr,0);
		if(substr(op,strlen(op)-4,-1)!="(a5)") continue;

		OpOffEx(addr,0,REF_OFF32,-1,jumptable,a5offs+2);

		jumpentry=jumptable-a5offs-2+GetOperandValue(addr,0);
		jumpresid=ltoa(Word(jumpentry+4),10);
		resoffs=LocByName("CODEResource"+jumpresid);
		funcoffs=resoffs+8+Word(jumpentry+0);

		AddCodeXref(addr,funcoffs,fl_CF|XREF_USER);
	}
}
```

# References #

  * http://developer.apple.com/legacy/mac/library/documentation/mac/MoreToolbox/MoreToolbox-99.html
  * http://developer.apple.com/legacy/mac/library/documentation/mac/pdf/Processes/Segment_Manager.pdf