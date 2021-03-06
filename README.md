<a href="https://dadess.cat/"><img src="input/dadess_logo.svg" width=300 title="Dadess" alt="Dades d'Economia Social i Solidària"></a>

# SSE activity in Catalunya during COVID-19

We analyze data for producers that have enabled the possibility to buy their product during the COVID-19 crisis.

New producers per region in Catalunya:
[![New projects map](http://g.recordit.co/8y9reUk99t.gif)]()


Regional logistics:
[![Logistics sankey](http://g.recordit.co/GUbagScowb.gif)]()


Payment methods of producer per dataset source:
[![Payment methods](http://g.recordit.co/yV8suyYMBH.gif)]()

## Getting started

In order to run the analysis notebook you will need to follow the next steps:

0. Setup a virtual environment
```
virtualenv --python=python3 venv
source venv/bin/activate
```
1. Install all the necesary libraries:
```bash {cmd}
pip install -r requirements.txt
```
2. Enable the notebook extensions so the Sankey diagrams are shown:
```bash {cmd}
jupyter nbextension enable --py --sys-prefix ipysankeywidget
```
3. If on virtual environment, add the kernel to jupyter
```
python -m ipykernel install --user --name=covid_agro
```
4. Start the notebook server from the command line:
```bash {cmd}
jupyter notebook
```
If the virtual environment was added as a kernel, choose it among the kernel list, just below the logout button.
5. Open `analysis.ipynb` on the web browser and run it. The second cell in the notebook will ask if you want to compute the pre-process of the input files. If it's the first time running the code you will need to input 'y' in the window that will appear, input 'n' otherwise.

## Documentation

### Summary

Schools, restaurants, food markets and other closed due to the COVID-19 pandemic. Farmers and agricultural and artesanal projects that usually provided those places with food did not have where to sell their product. The response of these projects was to enable a way for people to place orders (via telephone, online, etc.) and shipping to some areas.

Our <b>goal</b> was to analyze the data of some sources of Social Solidarity Economy (SSE) projects in Catalonia that were open as a response to the COVID crisis.

We compared two data sources of new projects with data from projects registered as local trade on the Government of Catalonia.

- [Arran de terra](https://www.arrandeterra.org/) and [Pam a Pam](https://pamapam.org/ca/): [Abastiment agroecològic](http://arrandeterra.org/abastiment/)
- [Unió de Pagesos](https://uniopagesos.cat/): [Pagesia a casa](https://pagesiaacasa.cat/)
- [Local trade producers](https://analisi.transparenciacatalunya.cat/en/Comer-/Productors-adherits-a-la-venda-de-proximitat/xmyy-7xqi)

The structured versions of the data can be found in [dadess open data platform](https://open.dadess.cat).

The analysis will consist on 3 main parts:
- A comparison between the data of the new projects and the local trade producers.
- Analysis of the new projects.
- Comparison between the new projects.


### Technical steps

#### Unification of data

Because the data is coming from different sources, we need to unify it. This entails different steps:

- <b>Text normalization</b>. Depending on the source, the data could contain typos, different upper and lowercase usage, accents, etc. In order to make the texts as consistent as possible we removed accents, lower and upper cases, tags, stopwords and special characters. We detected some typos in the locations and, because they were just a few cases, so we also correct the location typos.
- <b>Extract the location data on every file</b>. In some cases it is necessary to read multiple columns and even look in free text files for a location.
	1. We used a file with the different locations that exist in Catalonia to know which words to look for inside the data (using <b>exact matching</b> of words).
	2. We first look for locations (using the auxiliar location file) in the columns that were supposed to contain them.
	3. After that, we look for locations in the free text (specifically shipping location). Because we are interested in shipping locations, we used <b>regular expressions</b> to filter sentences where it looked like the shipping info could be informed.
- <b>Add missing columns</b>. Some sources had less input columns but some of the missing information could be retrieved from free text fields. We add a payment type column to the data that does not contain one extracting the data from the free text fields using <b>regular expressions</b>.
- <b>Create new columns</b>. In order to analyze some interesting data, we create new columns based on data from another columns:
	- Binary columns. Columns with a 1 when project matches the specified category or filter by the column name and 0 when it does not. Some of these columns indicate whether the producer sells meat or not (same with fruit, vegetables and other categories), whether it has a specific payment method (or combination of payment methods available) or not.
	- Count columns. These columns contain numbers representing the number of main product a producer sells (defyning main as meat, vegetables or fruits), other products, total products, payment methods and regions with delivery available.
- <b>Detect duplicates</b>. For the two data sources of new projects we need to check if some producers/brands are present in both of them. It is also interesting to know if any of the new projects were also registered as local trade before. We use <b>exact and fuzzy matching</b> to detect name duplicates. In the case of the fuzzy matching we take into account two different metrics, the ratio and the partial ratio:
	```
	fuzz.ratio("Social Solidarity Economy", "Social Solidarity Economics")
	> 96
	fuzz.ratio("Social Solidarity Economy (SSE)", "SSE")
	> 18
	fuzz.partial_ratio("Social Solidarity Economy (SSE)", "SSE")
	> 67
	```
	When a duplicate is found between the two datasets with new projects data, we will keep the information pertaining to the dataset with the most complete info. 
	When a new project has a duplicate in the local trade dataset, we will only place a 1 in a new column.

At the end of all this steps, we will have a dataset with the information with both sources with new projects with new columns.

To help with the representations later, we also use the library `geopy` to obtain the latitude and longitud of each region in Catalunya. 

#### Analysis

Catalonia is divided in 4 provinces (Barcelonès, Gironès, Lleida, Tarragonès) and can also be divided in 42 comarques (regions). Based on resources, these regions can be more prone to agriculture, livestock, wine production, etc. Thus, we decided to do the majority of analysis at a region level.

We ended up using 3 different plots:
- [Representation of a numeric value per region on a map.](#map-with-points-in-each-region)
- [Sankey diagrams.](#sankey-diagram-to-show-the-regional-logistics)
- [Bar plots.](#bar-plots-to-compare-producers)

##### Map with points in each region
[![New projects map](http://g.recordit.co/8y9reUk99t.gif)]()

A quicker way to plot data in a map without having to look for the geometric data of the desired region is to use a map image.
Using the libraries `plotly` and `Image`:
- We import a map of Catalunya in an image format. The positions of the image will correspond to max and min latitude and longitude of the map.
- Compute the number per comarca that we want to plot over the map.
- Plot the numbers per comarca with a circle with a normalized size.

##### Sankey diagram to show the regional logistics
[![Logistics sankey](http://g.recordit.co/GUbagScowb.gif)]()

Some of the producers we're analyzing have opened up the possibility of shipping to other regions and so we wanted to see what movements of product existed between regions. To do so we decided to show the data in a Sankey diagram. To read these, it is important to know some points:
- In these diagrams, the regions on the left will be the regions of origin and the regions on the right the regions where the products are delivered.
- If there is a line that goes from a region on the left to a region on the right it's because there are some producers from the first that deliver to the latter.
- The width of the region itself (not the line between regions) is directly proportional to the number of producers in said region (i.e. a region with a wider line will contain more producers than a region with a smaller one).
- The depth of color of the line between regions is directly proportional to the number of producers that deliver from the region of origin of the line to the region of delivery of the line (i.e. a darker line will mean more producers with that delivery "route").

These diagrams are plotted using the libraries `floweaver` and `ipysankeywidget`.

##### Bar plots to compare producers

[![Payment methods](http://g.recordit.co/yV8suyYMBH.gif)]()





