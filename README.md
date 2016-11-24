# SPRING

SPRING is a tool for exploring topological relationships in single-cell sequencing data that relies on interactive visualization of a force-directed graph. In the SPRING graph, each node is a cell and each edge is an instance of proximity in gene expression space. Users can upload their own expression data to SPRING using our webserver https://kleintools.hms.harvard.edu/tools/spring.html. However, heavy users are encouraged to create a local installation of SPRING (see below). 

## Quick start ##

#### Installing SPRING locally ####

2. Download the SPRING repo: go to the green "Clone or download" button on this page
2. Alternatively: Install git and in the terminal enter `git clone https://git@github.com/AllonKleinLab/SPRING.git`

#### Explore a processed dataset ####

1. Go to the SPRING directory by entering `cd SPRING`
2. Start a local server by entering `python -m SimpleHTTPServer 8000 &`
3. In a web browser (preferably Chrome) go to <a href="http://localhost:8000/springViewer.html?datasets/centroids">http://localhost:8000/springViewer.html?datasets/centroids</a>.

#### Process your own dataset ####

_To load your own data, use the `helper_functions.py` module. Detailed documentation for each helper function is given below. You can get started testing these functions on example input data by:_

1. Unzip `example_inputs/E.npy.zip`
2. In the SPRING directory, run the following pyhon code

            import pickle, numpy as np

            # Import SPRING helper functions
            from helper_functions import *

            # Import expression matrix; rows are cells and columns are genes
            ### ****** Make sure E.npy is unzipped *************
            print 'Loading expression matrix'
            E = np.load('example_inputs/E.npy')

            # Filter out cells with fewer than 1000 UMIs
            print 'Filtering cells'
            E,cell_filter = filter_cells(E,1000)

            # Normalize gene expression data
            print 'Row-normalizing'
            E = row_normalize(E)

            # Filter genes with mean expression < 0.1 and fano factor < 3
            print 'Filtering genes'
            _,gene_filter = filter_genes(E,0.1,3)

            # Z-score the gene-filtered expression matrix and do PCA with 20 pcs
            print 'Zscoring and PCA'
            Epca = get_PCA(Zscore(E[:,gene_filter]),20)

            # get euclidean distances in the PC space
            print 'Getting distance matrix'
            D = get_distance_matrix(Epca)

            # load additional data
            # gene_list is a list of genes with length E.shape[1]
            # cell_groupings is a dict of the form: { <grouping_name> : [<cell1_label>, <cell2_label>,...] }
            # a "grouping" could be the sample id, cluster label, or any other categorical variable
            gene_list, cell_groupings = pickle.load(open('example_inputs/additional_data.p'))

            # save a SPRING plots with k=5 edges per node in the directory "datasets/frog/"
            print 'Saving SPRING plot'
            save_spring_dir(E,D,5,gene_list,'datasets/frog', cell_groupings=cell_groupings)

3. If you haven't already, start a local server by entering `python -m SimpleHTTPServer 8000 &`
4. In a web browser, go to <a href="http://localhost:8000/springViewer.html?datasets/frog">http://localhost:8000/springViewer.html?datasets/frog</a>.

## User manual ##

#### Run SPRING: General instructions ####

1. Download the SPRING repo and `cd` into it (see "Installing SPRING locally" above)
2. Create a project directory and populate with your own data files (see "Create your own project" below)
2. Start a local server by entering  `python -m SimpleHTTPServer 8000 &`
3. Go to the following URL, which must be modified with the name of your project <a href="">http://localhost:8000/springViewer.html?PATH_TO_YOUR_PROJECT_DIRECTORY</a>.
       
#### Create your own project   

The SPRING project directory must contain files with stereotyped names and formats. Matlab and Python funtions are provided to create these files (see "Helper functions documentation" below). Here is a guide to the file names and formats:
 
1. **gene_colors/color_data_all_genes-*.csv [REQUIRED]** <br>
In a directory called `gene_colors` there must be (at most 50) base-0 numbered files called `color_data_all_genes-*.csv.` e.g.
 
            color_data_all_genes-0.csv
            color_data_all_genes-1.csv
            color_data_all_genes-2.csv
            ...
            color_data_all_genes-50.csv
Each of these files should contain gene expression for a subset of genes, with one gene on each row. The rows have the following format:
`GENE_NAME,cell1_expression,cell2_espression...`. For example, `Sox2,0.3,0.54,0.6... `. So if the dataset has `n` cells, this file should contain `n+1` columns. NOTE: Make sure that the file has no header, and that there are no extra commas on a line. 

2. **graph_data.json [REQUIRED]** <br>
Json file containing the graph data, with the following form (use base-0 numbering; any json compatibl format is OK):
 
            {  "nodes": [ {   "name": cell0, "number": 0 },     // List of nodes
                          {   "name": cell1, "number": 1 },
                          ....
                          {   "name": cellN, "number": N } ],
               "links": [ { "source": 10, "target": 23 },      // List of edges
                          { "source": 29, "target": 50 },
                          ....
                          { "source": 40, "target": 125 }  ] }

3. **color_stats.json [REQUIRED]** <br>'
Json file containing pre-calculated summary statistics of the various coloring tracks, including those in `color_data_gene_sets.csv` (see below) and `color_data_all_genes-*.csv`. The file should contain a dictionary mapping each color track-name to a list  [MEAN, STANDARD DEVIATION, MIN, MAX, 99-PERCENTILE] with summary statistics for that color track. Thus, this file could have the form:

            {  "Sox2":  [ 0.1, 0.2, 0, 1.46, 1.22],
               "Brca":  [ 5.2, 4.1, 0, 20.3, 18.1],
               ...
               "Gata1": [ 0.4, 0.3, 0, 5.42, 4.11]  }

4. **categorical_coloring_data.json [OPTIONAL]** <br>' 
Blah

5. **color_data_gene_sets.csv [OPTIONAL]** <br>' 
Blah

