Title: Automation Using Makefiles
date: 2018-03-15 06:00
authors: Matthew Kudija
comments: true
slug: makefile-automation
tags: python, makefile, make, automation

<!-- PELICAN_BEGIN_SUMMARY -->

Makefiles provide an easy way to group multiple terminal commands into a single command using `make <command>`. This gives a brief overview with just enough information to write your first Makefile.

<!-- PELICAN_END_SUMMARY -->


# Why Make?

Apparently Make was originally developed to build executable programs, but I have seen an [number](https://github.com/mwilliamson/python-makefile/blob/master/makefile) of [examples](https://github.com/getsentry/raven-python/blob/master/Makefile) of people using Make to manage Python workflows. In this example, let's say we have a Python script in several repositories that we use to update files in each respective repository. From the command line, we can easily `cd` into each and `python build.py` to run the update script and then commit the changes. Using a Makefile, however, we can automate all of that into a single command: `make publish`. Let's take a look at how to do this.


# Writing a Makefile
To start, our entire Makefile is below (and available for download [here](https://github.com/mkudija/blog/blob/master/content/downloads/code/Makefile)):. 

```
PY?=python3


help:
	@echo '                                                                          '
	@echo 'Makefile to update websites on MAG GitHub pages                           '
	@echo '                                                                          '
	@echo 'Usage:                                                                    '
	@echo '   make data                MILE_file_copy and update_utilization         '
	@echo '   make html                regenerate the websites                       '
	@echo '   make publish             regenerate websites, then publish to GitHub   '
	@echo '                                                                          '

data:
	python ../MILE_Data/MILE_file_copy.py
	cd ../Aircraft-Data/Utilization/Scripts; python update_utilization.py

html: data
	cd ../CommercialData/scripts; python build.py
	cd ../MarketView/scripts; python build.py
	cd ../FleetView/scripts; python build.py

publish: html
	@echo ' '
	cd ../CommercialData; git pull origin master
	cd ../CommercialData; git add -A
	cd ../CommercialData; git commit -m "auto-regenerate from make"
	cd ../CommercialData; git push origin master

	@echo ' '
	cd ../MarketView; git pull origin master
	cd ../MarketView; git add -A
	cd ../MarketView; git commit -m "auto-regenerate from make"
	cd ../MarketView; git push origin master

	@echo ' '
	cd ../FleetView; git pull origin master
	cd ../FleetView; git add -A
	cd ../FleetView; git commit -m "auto-regenerate from make"
	cd ../FleetView; git push origin master
```


Let's break this down to understand how to construct a Makefile. A Makefile has this basic structure (see the complete [Make documentation, particularly section 2.1](https://www.gnu.org/software/make/manual/make.html)):

```
target … : prerequisites …
        recipe
        …
        …
```

- **`target`** is name of the make command we are constructing. In our Makefile, `data: ...` means we can enter the command `make data` from the command line.
- **`prerequisites`** are targets to call before calling our `recipe`. In our Makefile, when we call `make publish`, `make html` will be automatically be executed first because it is a prerequisite.
- **`recipe`** is the group of commands our make command performs. Each line is a command, but commands can be given on the same line separated by `;`. When we call `make data` in our Makefile, the first command run is executing `python ../MILE_Data/MILE_file_copy.py`, and so on until all commands are completed.


We can now define Make commands, group tasks we want to perform, and define prerequisites as necessary. As with anything there is much more to learn about Makefiles, but this is enough to get you started. 

## A Note on Directories

The `cwd` of Python when run from a Makefile will by default be the directory that contains the Makefile. This may be an issue if you are running Python scripts that assume a `cwd` where they are located. To address this, you can enter the appropriate directory and execute the script on the same line in the Makefile, for example: `cd ../MarketView/scripts; python build.py`


# Executing Makefiles

Back to our example, the previous process required us to:

- `cd` into each repo
- run `python build.py` update files
- commit the changes
- repeat for each

Using this Makefile, the process now is simply:

- `cd` in to the directory containing `Makefile`
- `make publish`


# Conclusion

Whenever you find yourself doing the same rote task often it may be a good time to ask yourself if some simple automation is appropriate. For anything you perform at the command line, Makefiles can be a lightweight automation solution.