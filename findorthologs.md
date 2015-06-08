---
layout: default
title: findorthologs
---
<H1>FindOrthologs Tool</H1>
<HR>
<div class="Instructions">
<H5>Search for multispecies orthologs</H5>
    <p>
    Tables are created by reference to individual human genes.</p>
    <p>
    <b></b>Enter a  human proteome Uniprot Accession to pull a table for orthologs to that gene, ex. P00395</b>
    </p>
</div> 


<script type="text/javascript">
function changeText3(){
    var userInput = document.getElementById('userInput').value;
    var lnk = document.getElementById('lnk');
    lnk.href = "https://sheet.zoho.com/view.do?url=https://raw.githubusercontent.com/clairemcwhite/QfOreferencetables/master/" + userInput +".csv";
    lnk.innerHTML = lnk.href;
    window.open("https://github.com/clairemcwhite/QfOreferencetables/blob/master/" + userInput +".csv");
}
</script>

<input type='text' id='userInput' value='' />
<input type='button' onclick='changeText3()' value='Go'/>
<p>
Link to Excel view of most recent search: <a href="" id=lnk></a><br>
</p>
<br>

<HR>

<div class="More">
<h5>About</h5>
 
    <p>FindOrthologs displays multispecies orthologs to a reference human gene according to diverse ortholog sorting algorithms.</p>
    </p>
     <p>
    Tables indexed by UniProt Accession are currently available for the <a href="https://raw.githubusercontent.com/clairemcwhite/clairemcwhite.github.io/master/humanProteomeDetails.tsv">20,188 human protein coding genes</a> from the 2015 release of the UNIPROT reviewed human proteome ("UP000005640").  

    <p>
    
    Tables are created by searching for a gene within ortholog databases, and listing genes named as orthologs to it according to each database.  Table rows contain putative orthologs, labeled by Uniprot Accession, Uniprot Gene Name, and number of databases where that gene was called as an ortholog to the query gene.  Table column headers contain the database name and the ortholog group ID from that database. A gene which is called as an ortholog by a database will have a "1" in that database's column.   
    </p>    
   
</div> 

<div class="currentdatabases">

Ortholog groups are pulled from the <a href="http://orthology.benchmarkservice.org/cgi-bin/gateway.pl?f=ShowProject">Quest for Orthologs reference proteome benchmarking</a>.  The following algorithms ortholog calling algorithms were run on identical data sets.
<ul>
  
  <li>Hieranoid 2.0 (KO) release 74</li>
  <li><a href="http://www.ensembl.org/index.html">EnsemblCompara v2</a></li>
  <li><a href="http://inparanoid.sbc.su.se/cgi-bin/index.cgi">InParanoid</a></li>
  <li><a href="http://inparanoid.sbc.su.se/cgi-bin/index.cgi">InParanoidCore</a></li>
  <li><a href="http://orthology.phylomedb.org/">metaPhOrs</a> missing genomes: {PYRKO,STRCO,THEMA</li>
  <li><a href="http://omabrowser.org/oma/home/">OMA GETHOGs</a></li>
  <li><a href="http://omabrowser.org/oma/home/">OMA Groups (RefSet5)</a>OMA </li>
  <li><a href="http://omabrowser.org/oma/home/">OMA OMA Pairs (Refset5)</a></li>
  <li><a href="http://www.lbgi.fr/orthoinspector/">orthoinspector 1.30</a></li>
  <li><a href="http://pantherdb.org/">PANTHER 8.0 (all)</a></li>
  <li><a href="http://pantherdb.org/">PANTHER 8.0 (LDO only)</a></li>
  <li><a href="http://phylomedb.org/">phylomeDB</a></li>
  <li>RBH - 26 genomes missing</li>
  <li>RSD 0.8 1e-5 Deluca (Roundup)</li>
</ul>
    
The current proteome set covers <a href="http://www.ebi.ac.uk/reference_proteomes">66 species</a>, including eukaryotes, archaea and bacteria. 
</div>
<HR>
<div class="More">

    <p>
        

    <h5>Table viewing instructions</h5>
    Github initially renders a ".csv" file as an interactive table.  To save a table, click the "Raw" icon, and then save the page.  The <a href="https://github.com/clairemcwhite/OrthologLinking">entire collection of tables</a> can be downloaded as a zip file. 
    FindOrthologs gives a comprehensive view of multiorganism orthologs of a given human gene, as produced by diverse ortholog sorting algorithms. For example, the table for the human gene P00395 has columns containing the membership of the ortholog group P00395 belongs to according to each database.
<HR>
    <h5>Rationale</h5>

    Orthologous genes are genes which arise from speciation, for example, Human Talin, and Mouse Talin. Identification and accuracy of genes classed as orthologs is key for comparative genomic approaches, however there is no consensus model. Approximately 30 independent algorithms exist to classify orthologous genes; some based on species and gene phylogeny and some using exclusively sequence comparison. Phylogeny based approaches have the advantage of tracking genes through realistic evolutionary paths, however tend to be computationally intensive, and subject to error from misconstructed gene trees. BLAST approaches are much faster, generally using an all-by-all blast of multispecies proteomes in order to generate best matches, however lack the added power of phylogenetic information.

    There is often no clear answer for which database to use to generate lists of orthologs for a comparative project. Confounding this problem is the lack of a method to easily compare results from different databases. Additionally, precomputed data available from the databases use various gene naming methods, or different releases of the same naming type with non-overlapping gene names. FindOrthologs attempts to inform research using orthologs by standardizing and aligning ortholog groups from ten separate ortholog grouping algorithms. Additionally, the number of databases which call a gene as an ortholog may be used as a proxy for confidence of that gene's assignment of orthology to the reference gene. The ability to visualize and compare ortholog groupings from multiple sources will aid comparative study of proteins and genes.    

    </p>
</div>     
    


<div class="posts">
  {% for post in paginator.posts %}
  <article class="post">
    <h1 class="post-title">
      <a href="{{ site.baseurl }}{{ post.url }}">
        {{ post.title }}
      </a>
    </h1>

    <time datetime="{{ post.date | date_to_xmlschema }}" class="post-date">{{ post.date | date_to_string }}</time>

    {{ post.content }}
  </article>
  {% endfor %}
</div>
<HR>
<div class="source">

    <p>
Page template : <a href="https://github.com/barryclark/jekyll-now">Jekyll-now</a> 
    </p>
</div>     
    