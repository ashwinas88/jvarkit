# BioAlcidae


## Usage

```
Usage: bioAlcidae [options] Files
  Options:
    -e, --expression
      Javascript expression
    -F, --format
      force format: one of VCF BAM SAM FASTQ FASTA BLAST DBSNP. BLAST is BLAST 
      XML version 1. DBSNP is XML output of NCBI dbsnp. INSDSEQ is XML output 
      of NCBI EFetch rettype=gbc.
    -h, --help
      print help and exits
    -J, --json
      Optional. Reads a JSON File using google gson 
      (https://google-gson.googlecode.com/svn/trunk/gson/docs/javadocs/index.html 
      ) and injects it as 'userData' in the javascript context.
    -o, --output
      Output file. Optional . Default: stdout
    -f, --scriptfile
      Javascript file
    --version
      print version and exits

```


## Description

javascript version of awk for bioinformatics

## Compilation

### Requirements / Dependencies

* java compiler SDK 1.8 http://www.oracle.com/technetwork/java/index.html (**NOT the old java 1.7 or 1.6**) . Please check that this java is in the `${PATH}`. Setting JAVA_HOME is not enough : (e.g: https://github.com/lindenb/jvarkit/issues/23 )
* GNU Make >= 3.81
* curl/wget
* git
* xsltproc http://xmlsoft.org/XSLT/xsltproc2.html (tested with "libxml 20706, libxslt 10126 and libexslt 815")


### Download and Compile

```bash
$ git clone "https://github.com/lindenb/jvarkit.git"
$ cd jvarkit
$ make bioAlcidae
```

The *.jar libraries are not included in the main jar file, so you shouldn't move them (https://github.com/lindenb/jvarkit/issues/15#issuecomment-140099011 ).
The required libraries will be downloaded and installed in the `dist` directory.

### edit 'local.mk' (optional)

The a file **local.mk** can be created edited to override/add some definitions.

For example it can be used to set the HTTP proxy:

```
http.proxy.host=your.host.com
http.proxy.port=124567
```
## Source code 

https://github.com/lindenb/jvarkit/tree/master/src/main/java/com/github/lindenb/jvarkit/tools/bioalcidae/BioAlcidae.java

## Contribute

- Issue Tracker: http://github.com/lindenb/jvarkit/issues
- Source Code: http://github.com/lindenb/jvarkit

## License

The project is licensed under the MIT license.

## Citing

Should you cite **bioAlcidae** ? https://github.com/mr-c/shouldacite/blob/master/should-I-cite-this-software.md

The current reference is:

http://dx.doi.org/10.6084/m9.figshare.1425030

> Lindenbaum, Pierre (2015): JVarkit: java-based utilities for Bioinformatics. figshare.
> http://dx.doi.org/10.6084/m9.figshare.1425030



Bioinformatics file javascript-based reformatter ( java engine http://openjdk.java.net/projects/nashorn/ ). Something like awk for VCF, BAM, SAM, FASTQ, FASTA etc...


The program injects the following variables:


 *  out a java.io.PrintWriter ( https://docs.oracle.com/javase/7/docs/api/java/io/PrintWriter.html ) for output
 *  FILENAME a string, the name of the current input
 *   format a string, the format of the current input ("VCF"...)






#### VCF

For VCF , the program injects the following variables:

 *  header a htsjdk.variant.vcf.VCFHeader https://samtools.github.io/htsjdk/javadoc/htsjdk/htsjdk/variant/vcf/VCFHeader.html
 *  iter a java.util.Iterator<htsjdk.variant.variantcontext.VariantContext>  https://samtools.github.io/htsjdk/javadoc/htsjdk/htsjdk/variant/variantcontext/VariantContext.html
 *  vep a com.github.lindenb.jvarkit.util.vcf.predictions.VepPredictionParser  https://github.com/lindenb/jvarkit/blob/master/src/main/java/com/github/lindenb/jvarkit/util/vcf/predictions/VepPredictionParser.java
 *  snpeff a com.github.lindenb.jvarkit.util.vcf.predictions.AnnPredictionParser  https://github.com/lindenb/jvarkit/blob/master/src/main/java/com/github/lindenb/jvarkit/util/vcf/predictions/AnnPredictionParser.java





#### Fasta


 *  iter a java.util.Iterator<Fasta>





```

	public class Fasta 
		{
		public String getSequence();
		public String getName();
		public void print();
		public int getSize();
		public char charAt(int i);
		}

```






#### BLAST


 *  iter a java.util.Iterator<gov.nih.nlm.ncbi.blast.Hit> . gov.nih.nlm.ncbi.blast.Hit is defined by the Blast Document type definition (DTD). This iterator has also a method getIteration() that returns the following interface:

```
interface BlastIteration {
		public int getNum();
		public String getQueryId();
		public String getQueryDef();
		public int getQueryLen();
		}
	}
```


	





#### INSDSEQ


 *  iter a java.util.Iterator<gov.nih.nlm.ncbi.insdseq.INSDSeq> . gov.nih.nlm.ncbi.insdseq.INSDSeq is defined by the INSDSeq Document type definition (DTD).






#### XML


 *  iter a java.util.Iterator<gov.nih.nlm.ncbi.dbsnp.Rs> . gov.nih.nlm.ncbi.dbsnp.Rs is defined by the XSD schema http://ftp.ncbi.nlm.nih.gov/snp/specs/docsum_current.xsd






#### BAM or SAM



 *  header a htsjdk.samtools.SAMFileHeader http://samtools.github.io/htsjdk/javadoc/htsjdk/htsjdk/samtools/SAMFileHeader.html
 *  iter a htsjdk.samtools.SAMRecordIterator  https://samtools.github.io/htsjdk/javadoc/htsjdk/htsjdk/samtools/SAMRecordIterator.html






#### FASTQ


 *  iter a java.util.Iterator<htsjdk.samtools.fastq.FastqRecord>  https://samtools.github.io/htsjdk/javadoc/htsjdk/htsjdk/samtools/fastq/FastqRecord.html






### Example



#### BAM

getting an histogram of the length of the reads


```

L={};
while(iter.hasNext()) {
	var rec=iter.next();
	if( rec.getReadUnmappedFlag() || rec.isSecondaryOrSupplementary()) continue;
	var n= rec.getReadLength();
	if(n in L) {L[n]++;} else {L[n]=1;}
	}
for(var i in L) {
	out.println(""+i+"\t"+L[i]);
	}

```





#### Fasta

"Creating a consensus based on 'x' number of fasta files" ( https://www.biostars.org/p/185162/#185168)



```

$ echo -e ">A_2795\nTCAGAAAGAACCTC\n>B_10\nTCAGAAAGCACCTC\n>C_3\nTCTGAAAGCACTTC" |\
java -jar ~/src/jvarkit-git/dist/bioalcidae.jar -F fasta -e 'var consensus=[];while(iter.hasNext()) { var seq=iter.next();out.printlnseq.name+"\t"+seq.sequence);for(var i=0;i< seq.length();++i) {while(consensus.length <= i) consensus.push({}); var b = seq.charAt(i);if(b in consensus[i]) {consensus[i][b]++;} else {consensus[i][b]=1;} } } out.print("Cons.\t"); for(var i=0;i< consensus.length;i++) {var best=0,base="N"; for(var b in consensus[i]) {if(consensus[i][b]>best) { best= consensus[i][b];base=b;}} out.print(base);} out.println();' 


A_2795      TCAGAAAGAACCTC
B_10         TCAGAAAGCACCTC
C_3           TCTGAAAGCACTTC
Cons.        TCAGAAAGCACCTC

```





#### VCF

Reformating a VCF
we want to reformat a VCF with header


```

CHROM POS REF ALT GENOTYPE_SAMPLE1 GENOTYPE_SAMPLE2 ... GENOTYPE_SAMPLEN

```


we use the following javascript file:



```

var samples = header.sampleNamesInOrder;
out.print("CHROM\tPOS\tREF\tALT");
for(var i=0;i< samples.size();++i)
	{
	out.print("\t"+samples.get(i));
	}
out.println();

while(iter.hasNext())
	{
	var ctx = iter.next();
	if(ctx.alternateAlleles.size()!=1) continue;
	out.print(ctx.chr +"\t"+ctx.start+"\t"+ctx.reference.displayString+"\t"+ctx.alternateAlleles.get(0).displayString);
	for(var i=0;i< samples.size();++i)
		{
		var g = ctx.getGenotype(samples.get(i));

		out.print("\t");

		if(g.isHomRef())
			{
			out.print("0");
			}
		else if(g.isHomVar())
			{
			out.print("2");
			}
		else if(g.isHet())
			{
			out.print("1");
			}
		else
			{
			out.print("-9");
			}
		}
	out.println();
	}

```






```

$ curl -s  "ftp://ftp.1000genomes.ebi.ac.uk/vol1/ftp/release/20130502/ALL.chr22.phase3_shapeit2_mvncall_integrated_v5a.20130502.genotypes.vcf.gz" | \
gunzip -c | java -jar ./dist/bioalcidae.jar -f jeter.js -F vcf | head -n 5 | cut -f 1-10

CHROM	POS	REF	ALT	HG00096	HG00097	HG00099	HG00100	HG00101	HG00102
22	16050075	A	G	0	0	0	0	0	0
22	16050115	G	A	0	0	0	0	0	0
22	16050213	C	T	0	0	0	0	0	0
22	16050319	C	T	0	0	0	0	0	0

```



***
for 1000 genome data, print CHROM/POS/REF/ALT/AF(europe):



```

$ curl  "ftp://ftp.1000genomes.ebi.ac.uk/vol1/ftp/release/20130502/ALL.wgs.phase3_shapeit2_mvncall_integrated_v5a.20130502.sites.vcf.gz" |  gunzip -c |\
java -jar dist/bioalcidae.jar  -F VCF -e 'while(iter.hasNext()) {var ctx=iter.next(); if(!ctx.hasAttribute("EUR_AF") || ctx.alternateAlleles.size()!=1) continue; out.println(ctx.chr+"\t"+ctx.start+"\t"+ctx.reference.displayString+"\t"+ctx.alternateAlleles.get(0).displayString+"\t"+ctx.getAttribute("EUR_AF"));}' 

1	10177	A	AC	0.4056
1	10235	T	TA	0
1	10352	T	TA	0.4264
1	10505	A	T	0
1	10506	C	G	0
1	10511	G	A	0
1	10539	C	A	0.001
1	10542	C	T	0
1	10579	C	A	0
1	10616	CCGCCGTTGCAAAGGCGCGCCG	C	0.994
(...)

```





#### Blast




```
$ cat ~/input.blastn.xml | java -jar dist/bioalcidae.jar -F blast -e 'while(iter.hasNext())
 	{
 	var query  = iter.getIteration();
 	var hit = iter.next();
 	out.println(query.getQueryDef()+" Hit: "+hit.getHitDef()+"  num-hsp = "+hit.getHitHsps().getHsp().size());
 	}'

```



output:



```
$
Enterobacteria phage phiX174 sensu lato, complete genome Hit: Escherichia coli genome assembly FHI90 ,scaffold scaffold-6_contig-25.0_1_5253_[organism:Escherichia  num-hsp = 2
Enterobacteria phage phiX174 sensu lato, complete genome Hit: Acinetobacter baumannii AC12, complete genome  num-hsp = 2
Enterobacteria phage phiX174 sensu lato, complete genome Hit: Escherichia coli genome assembly FHI7 ,scaffold scaffold-5_contig-23.0_1_5172_[organism:Escherichia  num-hsp = 2
Enterobacteria phage phiX174 sensu lato, complete genome Hit: Escherichia coli genome assembly FHI92 ,scaffold scaffold-6_contig-18.0_1_5295_[organism:Escherichia  num-hsp = 2
Enterobacteria phage phiX174 sensu lato, complete genome Hit: Amycolatopsis lurida NRRL 2430, complete genome  num-hsp = 2
Enterobacteria phage phiX174 sensu lato, complete genome Hit: Escherichia coli genome assembly FHI87 ,scaffold scaffold-4_contig-19.0_1_5337_[organism:Escherichia  num-hsp = 2
Enterobacteria phage phiX174 sensu lato, complete genome Hit: Desulfitobacterium hafniense genome assembly assembly_v1 ,scaffold scaffold9  num-hsp = 2
Enterobacteria phage phiX174 sensu lato, complete genome Hit: Escherichia coli genome assembly FHI79 ,scaffold scaffold-4_contig-23.0_1_3071_[organism:Escherichia  num-hsp = 1
Enterobacteria phage phiX174 sensu lato, complete genome Hit: Escherichia coli genome assembly FHI24 ,scaffold scaffold-8_contig-33.0_1_3324_[organism:Escherichia  num-hsp = 2
Enterobacteria phage phiX174 sensu lato, complete genome Hit: Escherichia coli genome assembly FHI89 ,scaffold scaffold-8_contig-14.0_1_3588_[organism:Escherichia  num-hsp = 2
Enterobacteria phage phiX174 sensu lato, complete genome Hit: Sphingobacterium sp. PM2-P1-29 genome assembly Sequencing method ,scaffold BN1088_Contig_19  num-hsp = 2
Enterobacteria phage phiX174 sensu lato, complete genome Hit: Escherichia coli genome assembly FHI43 ,scaffold scaffold-3_contig-14.0_1_2537_[organism:Escherichia  num-hsp = 1
(...)
```





#### NCBI Sequence INDSeq




```
$  curl "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=nucleotide&id=25,26,27&rettype=gbc" |\
java -jar dist/bioalcidae.jar  -F INSDSEQ -e 'while(iter.hasNext()) {var seq= iter.next(); out.println(seq.getINSDSeqDefinition()+" LENGTH="+seq.getINSDSeqLength());}'
```



output:



```
Blue Whale heavy satellite DNA LENGTH=422
Blue Whale heavy satellite DNA LENGTH=416
B.physalus gene for large subunit rRNA LENGTH=518
```





#### NCBI DBSNP


 

```
$  curl -s "ftp://ftp.ncbi.nih.gov/snp/organisms/human_9606/XML/ds_chMT.xml.gz" | gunzip -c |\
 java -jar dist/bioalcidae.jar  -F dbsnp -e 'while(iter.hasNext())
 	{ var rs= iter.next();
 	out.println("rs"+rs.getRsId()+" "+rs.getSnpClass()+" "+rs.getMolType());
 	}'

rs8936 snp genomic
rs9743 snp genomic
rs1015433 snp genomic
rs1029272 snp genomic
rs1029293 snp genomic
rs1029294 snp genomic
rs1041840 snp genomic
rs1041870 snp genomic
rs1064597 snp cDNA
rs1116904 snp genomic
(...)

```






### See also


 *  https://github.com/lh3/bioawk
 *  https://www.biostars.org/p/152016/
 *  https://www.biostars.org/p/152720/
 *  https://www.biostars.org/p/152820/
 *  https://www.biostars.org/p/153060/
 *  https://www.biostars.org/p/183197
 *  https://www.biostars.org/p/185162/#185168





