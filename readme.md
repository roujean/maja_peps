

# peps_maja
PEPS is the French Sentinel collaborative ground segment: https://peps.cnes.fr. PEPS is mirroring all the Sentinel data provided by ESA for Sentinel1 and 2 (L1C), and is providing a simplified access to those data sets. It also implements a few processors among which [MAJA atmospheric correction processor](http://www.cesbio.ups-tlse.fr/multitemp/?p=6203). This repository contains two scripts to submit a MAJA L2A processing and to download MAJA L2A products from PEPS. MAJA processing is implemented in two ways within PEPS :
- single product MAJA processing (for which an browser interface is also available)
- full time series MAJA processing (which is only available on the command line)

The full time series production, that we call, with our excellent Frenglish, "Full_MAJA", is much more efficient than the single product processing. If you try to produce a time series with the single product, as MAJA requires time series, the products will be processed several times, requiring a large processing power used inefficiently.

As a result, we provide here an interface to use full maja production on PEPS from python command lines.

- full_maja_process.py asks [PEPS](https://peps.cnes.fr) to process Sentinel-2 products with MAJA. 
- full_maja_download.py checks if the processing is completed and eventually downloads the products. 

This code is only the emerged part of an Iceberg project, which is the implementation of MAJA within PEPS, which was done by a colleague at CNES, Christophe Taillan.

This code relies on python (it should work with python 2 and 3). It was developped for linux, and I am not sure it can work on windows.


 

## Examples

Using the on-demand MAJA process, two python programs are needed, one to submit the processing, and one to check completion, and to retrive the data it it is finished. 
 
### full_maja_process

This script will ask PEPS to process a continuous time series (from a begin date to an end date). It is currently limited to a maximum of one year and a minimum of two months, it is also limited to the processing of 10 tiles in parallel  for all the users in the world. If this limit is reached, the process will be pending untin one of the processed ends. You will have to start a different command line for each tile you want to process. Because of issues with Sentinel-2 L1C products before the 1st of April 2016, full_maja processing must start after the 1st of April 2016. 

The command line is very simple :

 `python ./full_maja_process.py  -t 31TCJ -a peps.txt -d 2017-07-01 -e 2018-01-01 -l 31TCJ_20170101.log` 

It will submit Maja processing for the Sentinel-2 L1C products acquired in the last 6 months of 2017 above Toulouse (31TCJ tile). A log file is issued which is necessary to check the completion and download the products with full_maja_download.py. If you want to start several tiles in parallel, remember to specify a different logfile, or you will lose the information to retrieve the data. If you have done such an error (I know it will happen ;) ), it is still possible to retrieve the results from the PEPS interface after having logged.

The option `-a peps.txt` provides the authentification information (see below for its format)

*Specifying the relative orbit*

Sometimes, a given tile is fully covered by one orbit, and only partially by another one. To save processing time and space, you can select to process only one orbut by specifying the orbit obtion with -o option.

- `python ./full_maja_process.py  -t 31TCJ -o 51 -a peps.txt -d 2017-07-01 -e 2018-01-01 -l 31TCJ_20170101.log` 

It will only process the data from relative orbit 51. But if you want to process the two orbits for one tile, it is better to specify nothing than launching two separate processings with each orbit.



### full_maja_download

 - ` python full_maja_download.py -a peps.txt -l 31TCJ_20170101.log`
 
The syntax of full_maja_download.py is even simpler, only the authentification information and the log file are necessary.

 - ` python full_maja_download.py -a peps.txt -l 31TCJ_20170101.log -w /path/to/31TCJ`

You may also specify an output directory for downloading the datasets.

The processing of the time series with MAJA can't be done in parallel, as date D+1 needs the result of date D to be processed. It takes less than one hour to generate the first product of a time series, which is based on a complex initiatlisation procedure, and then around 25 minutes per date to process. To save time and space, the dates with more than 90% of clouds are not issued. The products are only made available at the end, when everything is produced, so if you are noat patient, you will have to start  the command line several times, but there is no need to check it every minute, givent the expected amount of time needed. The percentages provided in the log file are very exagerated, they are just usefull for yo to see that progress is being made in the processing.

### processing capacity
Two problem may cause the on demand processing with MAJA to be kept pending for a while :
- our processor uses the High Performance Computing center at CNES,named HAL, which is a powerful but busy center. If it is too busy, you will have to wait until processors get available. MAJA will be pending.

- It also relies on a robot (HPSS) that downloads the Sentinel-2 data stored on tapes. When HPSS is overloaded, it happens that some products are not provided in time and not processed resulting in data gaps in the time series. To prevent that we have limited the processing capacity to 10 tiles in parallel, for all users in the world. As a result, the process could be "pending" for a while too. It will mean our on-demand processing facility is a success ;) In case some data gaps occur because of the HPSS is overloaded, the missing products will be counted and listed.


## Old version
This script will ask PEPS to process a series of products based on a catalog request which can be made according to various criteria.  Then the processing is schedule. If more than 10 products are to be ordered, a confirmation is asked to the user, to avoid submitting erroneous requests to PEPS. 

- `python ./peps_maja_process.py  -l Toulouse -a peps.txt -d 2017-11-01 -f 2017-12-01 -p prod_list_toulouse.txt` 

will submit Maja processing for the Sentinel-2 L1C products acquired in November 2017 above Toulouse. When you provide a date YY-MM-DD, it is actually YY-MM-DD:00:00:00. So a request with `-d 2015-11-01 -f 2015-11-01` will yield no result, while `-d 2015-11-01 -f 2015-11-02` will yield data acquired on 2015-11-01 (provided they exist). The list of ordered L2A products is stored in prod_list_toulouse.txt


- `python ./peps_maja_process.py  --lon 1 --lat 43.5 -a peps.txt -d 2017-11-01 -f 2017-12-01 -p prod_list_toulouse.txt`

 which downloads the Sentinel-2 products above --lon 1 --lat 43.5 (~Toulouse), acquired in November 2017. The list of ordered L2A products is stored in prod_list_toulouse.txt
 
 - `python ./peps_maja_process.py 2 --lon 1 --lat 43.5 -a peps.txt -d 2017-11-01 -f 2017-12-01 -o 51 -p prod_list_toulouse.txt ` 

 which downloads the Sentinel-2 products above --lon 1 --lat 43.5 (~Toulouse), acquired in November 2017 from orbit path number 51 only.  The list of ordered L2A products is stored in prod_list_toulouse.txt

The processing at peps takes between one or two hours. For this reason, the dowload is asynchronous and handled by another software.

### peps_maja_download

 - `python peps_maja_download.py -p prod_list_toulouse.txt -a peps.txt -w /mnt/data/PEPS_MAJA'

checks the completion of the products listed in prod_list_toulouse.txt, and eventually download them to the specified directory /mnt/data/PEPS_MAJA

If a few products are not completed yet, you will have to try again later.




## Authentification 

The file peps.txt must contain your email address and your password on the same line, such as:

`your.email@address.fr top_secret`

To get an account : https://peps.cnes.fr/rocket/#/register


