[#][1] Nov 2, 2017

I was deploying one of our applications this week but ran into an error where
ansible told me `No space left on device`

This was new to me because the application is a very small Flask application,
hosted on ec2 with at least a couple gigs of storage. There was going to be no
way that it was actually full.

So I went into the machine and checked

[code]

    df -h
    
    Filesystem      Size  Used Avail Use% Mounted on
    udev            487M     0  487M   0% /dev
    tmpfs           100M  8.0M   92M   9% /run
    /dev/xvda1      7.8G  5.7G  1.7G  77% /
    tmpfs           496M     0  496M   0% /dev/shm
    tmpfs           5.0M     0  5.0M   0% /run/lock
    tmpfs           496M     0  496M   0% /sys/fs/cgroup
    tmpfs           100M     0  100M   0% /run/user/1000
    
[/code]

This sure doesn't look like a full disk to me, so I had to think about what
else it could be.

[code]

    for i in /usr/*; do count=`sudo find $i | wc -l`; if [ $count -gt 10000 ]; then echo $i $count; fi; done
    
[/code]

This shows that the culprit is the inodes. In my case, it was a bunch of
`linux-header` files that took up a ton of space. Make sure you don't delete
the actual linux-header you want, check with `uname -rv`.

Then you can run `apt-get autoremove`

Please enable JavaScript to view the [comments powered by Disqus.][2]
[comments powered by Disqus][3]

   [1]: http://billyfung.com

   [2]: http://disqus.com/?ref_noscript

   [3]: http://disqus.com

[#][1] Oct 5, 2017

## It's not a bug, it's a feature

So while poking around at a bug in one of our database views, I looked at
another view that queried it and happened to find some code like this:

[code]

    SELECT col1
    FROM right_table R
    WHERE r.col1 IN (
        SELECT DISTINCT r.col1
        FROM wrong_table
    );
    
[/code]

Where `wrong_table` doesn't have `col1`. Now this code has been in production
for months now, and no bugs were found from it and it all worked well. The
query above will select all rows of `right_table` unless `wrong_table` is
empty, in which case it selects no rows. This code gave no errors in
production because the table that the view is based on already filters for the
result, but that wasn't ideal.

[code]

    SELECT col1
    FROM right_table r
    WHERE r.col IN (
        SELECT w.col
        FROM wrong_table w
    );
    
[/code]

Please enable JavaScript to view the [comments powered by Disqus.][2]
[comments powered by Disqus][3]

   [1]: http://billyfung.com

   [2]: http://disqus.com/?ref_noscript

   [3]: http://disqus.com

[#][1] Jul 5, 2017

## Docker, Vagrant, Chef, Puppet, Kubernetes… etc

So when dealing with writing code that you don't have to act upon production
data, you use an environment that allows you to break things continuously
until you find something that works. This is the ideal development cycle, one
where you can break states of your program, and be able to revert quickly and
easily. All the tools listed above help in some form of that. When you're
developing something that will go up into production on a server somewhere,
you the ideal development environment is one that mirrors the production one,
so when you want to deploy to production, the process will be seamless.

I currently use [Vagrant][2] and [Ansible][3] mostly for this. With a recent
addition of using [packer][4] to build virtualbox images. I have used
[docker][5] in the past, but back then the toolchain was still not very
refined and it was a tough learning curve. These days it's gotten much better,
but I haven't had time to test it out.

I mostly use Amazon EC2 cloud servers, but that can be a whole different
discussion.

## But what if you're doing analysis?

For those that deal with not just web based services, but also have to do
analysis, how does this fit into the picture? Often I download datasets and
explore them a bit with R or Python until deciding on if the dataset is useful
at all. Up until now, I've been doing this all locally, since my thinking
never defaults to using cloud based technology first. But recently I've had to
deal with datasets that are larger than usually, 20-40gb of data. I know this
isn't really that much data, but my laptop gets a bit stuffed when trying to
deal with all this locally.

So I've been thinking that I should be doing analysis like this on the cloud,
since the analysis might lead to eventually having the data in production. But
then there is an intermediate task of being able to share the results of
analysis easily.

I've played around with setting up an EC2 instance with RStudio on it, using
[Louis Aslett's][6] RStudio AMI. This makes it incrediably easy to set up
RStudio that you can access via the web, and also let's you run potentially
long scripts on the cloud while you do other work. (I say this as I'm running
an image processing script locally that is taking hours…)

In the near future I plan on exploring this further and figuring out the ideal
workflow for doing analysis as well as displaying results on the web.
Something I've also been thinking about is turning processing scripts into AWS
Lambda functions that will output into a S3 bucket for displaying.

Stay tuned!

Please enable JavaScript to view the [comments powered by Disqus.][7]
[comments powered by Disqus][8]

   [1]: http://billyfung.com

   [2]: https://www.vagrantup.com/

   [3]: https://www.ansible.com/

   [4]: https://www.packer.io/

   [5]: https://www.docker.com/

   [6]: http://www.louisaslett.com/RStudio_AMI/

   [7]: http://disqus.com/?ref_noscript

   [8]: http://disqus.com

[#][1] Jul 25, 2017

# Why doesn't it work properly in the cloud?

So the situation is this, I have a list of N webpages I need to check. My
script visits each webpage with some authentication, and checks to see what
kind of files are available to download on that page. Then I go and download
each of those files on the page, usually there are more than 5.

This looks something like (terrible pseudo code written for fun): `
pages_with_files = [list of downloads] for page in pages_with_files: r =
requests.get(page) soup = BeautifulSoup(r.text) the page to look for links
soup.findall(a href blah blah .file) d = requests.get(file to download) with
open(file_I_wanted) as j: copyfileobj(d) save d somewhere `

The task itself is something I've done a gazillion times, nothing too
difficult or tricky about it. The only difference is that the size is a bit
larger than what I'm used to. This script was run locally first, resulting in
a >20gb scraping job.

## To the cloud

Apart from taking 3 hours to scrape, it also takes up valuable resources on my
computer, which I can't really be giving away. So I figured I'd toss this into
the Amazon AWS cloud and just have it all done there. A very straight forward
Python script, so it should be no problem right?

Except the script runs into intermittent network problems. Now instead of just
pressing go and going to grab a coffee, I have to worry about the script going
and then stopping.

The error: `('Connection aborted.', BadStatusLine("''",))`

Without spending too much time on it, this means that the targeted server
isn't responding, or is giving a weird response.

## How to fix this?

I actually didn't have too much time to find the bug, largely because
sometimes it would download thousands of files without an error, but sometimes
it would only download a couple hundred. So my solution was just to keep an
eye on it, and run it again.

One of my ideas was to change timeouts, perhaps it had to do with the cloud
server taking a longer route between servers.

Next would be to just keep trying on the failed one.

Moral: I didn't have time to fix it, too many other things to work on. I got
the files in the end, and got other work done in the meantime.

Please enable JavaScript to view the [comments powered by Disqus.][2]
[comments powered by Disqus][3]

   [1]: http://billyfung.com

   [2]: http://disqus.com/?ref_noscript

   [3]: http://disqus.com

[#][1] Sep 9, 2017

Continuing on from the discussion about needing beefier computation, sometimes
throwing more money and power at the problem isn't the answer. Well it rarely
is, and even though I prefer to take time to find more optimal and efficient
solutions, in the real world I only have so much time and resources to get a
problem solved.

## Following a procedure

Coming from a more classical statistics background, my view of the more recent
advances in machine learning is that I am always a bit behind in the newer
advancements, especially ones which require a more technical background to
approach. It is always best to start off with more basic (and fast) algorithms
to establish a baseline before diving deep into models that take hours to
train. Following a routine of basic linear regression first, and then basic
tree models, and then more complicated mixtures allows for easy to follow
analysis. This way, if you have a linear model that is quick to train that
performs better than more advanced models, you won't be wasting time.

The other benefit of trying out the older and more understood models is that
it helps to develop an intuition into your data. The dark art of model
building and choosing what fits best comes from practice and doing it over and
over again. Which brings me to the main topic of [LightGBM][2].

## A new challenger

XGBoost came into the scene awhile back and drew many followers through
winning Kaggle competitions. Without going too deep into the technical bits of
gradient boosted frameworks, XGboost makes usage of trees that run in parallel
to perform decisions.

With XGBoost, I used it within the h2o machine learning engine, which make it
easy to set up and test multiple models. But I quickly found that it became
very memory intensive, so I looked to other algorithms.

LightGBM offers better memory management, and a faster algorithm due to the
"pruning of leaves" to manage the number and depth of trees that are grown.

## R and LightGBM

### Compiler set up

[code]

    # for linux
    sudo apt-get install cmake
    # for os x
    brew install cmake
    brew install gcc --without-multilib
    
[/code]

Then to install LightGBM as a R-package "` git clone -recursive
https://github.com/Microsoft/LightGBM cd LightGBM/R-package

# export CXX=g++-7 CC=gcc-7 # for OSX

R CMD INSTALL -build . -no-multiarch "`

### Organising the dataset

As usual, each algorithm wants specific formats for the input data, usually in
the form of a matrix (sparse or not).

[code]

    categoricals = c('V1', 'V2', 'V3')
    
    dtrain <- lgb.Dataset((data.matrix(train[,x_features, with=FALSE])),
                         categorical_feature = categoricals,
                         label = train$target)
    
[/code]

With your dataset partitioned into train and test dataframes/datatables, the
simple way is to convert into a matrix. This will create a lgb dataset object
with the training data.

### Building the model

[code]

    system.time(
    model <- lgb.train(params,
                       dtrain,
                       nrounds = 5500,
                       num_leaves = 50,
                       learning_rate = 0.1,
                       max_bin = 550,
                       verbose = 0
                       )
    )
         user    system   elapsed
    24761.032     5.844  2107.373
    
    preds <- predict(model, data.matrix(test[,x_features, with=FALSE]))
    
[/code]

I have included the computation numbers here as a comparison for other models.
The training data is 33m rows, which ends up being the largest dataset I've
ever worked out. The size itself was the issue for me, which lead me to use
LightGBM.

## Comparison

A semi-tuned gbm model gave me RMSE results of around 4, which is the number
I've been working towards. The issue arose when I wanted to tune xgb models
with h2o, which took around 2 hours to train a model. Of course, there will be
a factor of me not knowing what I'm really doing. Being limited in
computational power, I didn't know how to properly set up a distributed xgb
model with h2o, which I'm not sure is even possible with the current versions.
This lead me to not be able to properly figure out what the optimal parameters
for the model are.

To me, LightGBM straight out of the box is easier to set up, and iterate.
Which makes it easier do tuning and iteration of the model. I can train models
on my laptop with 8gb of RAM, with 2000 iteration rounds. This I could've
never done with h2o and xgb.

Of course, there is still a lot of other tuning and investigation that needs
to be done, but that's for next time.

Please enable JavaScript to view the [comments powered by Disqus.][3]
[comments powered by Disqus][4]

   [1]: http://billyfung.com

   [2]: https://github.com/Microsoft/LightGBM

   [3]: http://disqus.com/?ref_noscript

   [4]: http://disqus.com

[#][1] Sep 24, 2017

## Known Pleasures

I apparently missed the whole twitter hype on "joyplots", aptly named after
the album cover of Unknown Pleasures.

![Unknown Pleasures][2]

Now there is a [CRAN package][3] to create these ridgeline plots, so everybody
can have some fun. These plots are useful to visualise distributions of data
to compare them on the same scale. Or you can just make some cool looking
chart that is actually very confusing.

![2][4]

![joyplo2][5]

![3][6]

Please enable JavaScript to view the [comments powered by Disqus.][7]
[comments powered by Disqus][8]

   [1]: http://billyfung.com

   [2]: https://i.ytimg.com/vi/wOlQJRpTohs/hqdefault.jpg

   [3]: https://cran.r-project.org/web/packages/ggridges/index.html

   [4]: /writing/2017/09/demand_ridges.png

   [5]: /writing/2017/09/joyplo2.png

   [6]: /writing/2017/09/ggjoy.png

   [7]: http://disqus.com/?ref_noscript

   [8]: http://disqus.com

[#][1] Sep 27, 2017

## 196 starters, 44 countries and 267.5km around Bergen

With another season of cycling gone by, another world championship race means
data to look at. Ok so I was a bit tired of doing work and saw that somebody
had scraped the data so I took a quick look into it. The world championship
race is a bit more interesting because the riders are grouped by country, and
some countries don't have as many riders in the race. Usually this means that
a country with more riders should be at an advantage to win.

![start][2]

This plot doesn't really show much except how dispersed most of the riders are
at the start line. All 44 countries being represented nicely.

Something I was most curious about is how the riders the top 3 finishing
nations moved throughout the race. If there was more granular data (more
checkpoints at which positions were taken) then it would make for a more
insightful look into how a rider races.

![top3][3]

I knew that Slovakia had a smaller team compared to Norway and Australia, so
this should place them at a disadvantage, especially when their leader is
marked as a favourite. And you can tell that the Slovak team had riders placed
pretty sparsely whenever they crossed the lap finish line. Whereas Australia
and Norway had their riders pretty close to the front as a group for a large
portion of the race.

If there was another checkpoint for position, then it would've been
interesting to see how much yoyoing between checkpoints each rider does. This
might be useful to see in terms of knowing which rider follows which attack,
and then trying to get a metric for how much effort a rider as spent. Riding
all day in the back of a pack takes less effort than being near the front and
following attacks and changes in speed.

![vasil][4]

I took a look at the top 20 finishers, and one rider that stood out was Vasil
Kiryienka. For most of the race he was at the back of the peloton as they
crossed the lap finish line, and then he very effectively moved up and
finished 15th. Kiryienka is from Belarus, a nation with 3 riders in the race,
so this meant that they probably rode solo, and maybe had 1 rider be the
waterbottle/food fetcher. You can play with this [interactive plot of top 20
finishers][5] here.

Along with that plot of top 20 finishers, I also have [one for all
nations][6], so you can take a look at how riders of each nation positioned
themselves. This will let you see if maybe Belarus had all their riders
together and then they helped Kiryienka move up, or not.

![packfodder][7]

Can't finish without a joyplot (ridge plot) right? This shows pretty
effectively that Canada and USA were just pack fodder in the race. Australia
and Norway had their team near the front pretty much constantly throughout the
race, while USA and Canada were more right skewed in distribution.

## Later analysis

That was a fun quick analysis on just position, but what about how positions
deviated through the laps? Or how about looking into what the distribution of
speeds were like based on position. Lots more to be looked at in this data,
but back to work I'm paid for.

Please enable JavaScript to view the [comments powered by Disqus.][8]
[comments powered by Disqus][9]

   [1]: http://billyfung.com

   [2]: /writing/2017/09/starting_pos.png

   [3]: /writing/2017/09/top3_country.png

   [4]: /writing/2017/09/vasil.png

   [5]: https://plot.ly/~bfung/184.embed

   [6]: https://plot.ly/~bfung/186.embed

   [7]: /writing/2017/09/packfodder.png

   [8]: http://disqus.com/?ref_noscript

   [9]: http://disqus.com

[#][1] Sep 19, 2017

## Small data big charts

All the hype these days has been about data science and machine learning and
how everybody can pick up those hot skills and land a high paying job. I'm not
saying that it's as easy as that, but often you have to know where to look,
and what you're looking for. A bit like gold panning during the gold rush…
Sometimes you might just hit jackpot.

I've been interviewing and recruiting a bit right now and one of the questions
I've been asked often is: "What do you do on a daily basis?" Quite a valid
question, and I had already thought about this beforehand while I was trying
to ponder what I might display on a screen to interest students. Often my job
is to find sources of vectors of data that might be a piece of the puzzle that
is the electricity market. And the first step of that process is getting the
data, and then looking at it.

[code]

    ggplot(data=demand[nzyear>=2009 ]) +
      geom_point(aes(nzday, avg, colour=node)) +
      facet_grid(nzyear~nzmonth) +
      theme(axis.text.x  = element_text(angle=90, vjust=0.5, size=7))
    
[/code]

I'm a big advocate of large screens because I usually like to lay everything
out to bring with, then start to whittle the data down into smaller pieces.
Not dissimilar to an artist starting off with a large block of marble before
chipping it down. A very useful tool for multi dimensional data is
`facet_grid()`, especially if it's time series data that you can break up into
years and months.

![chch][2]

In the chart above, I'm plotting electricity demand at certain nodes, and
looking for anything that might stand out. Signals that might allow me to
cluster the data in a certain way, or characterise a certain feature.

## August 2012

First thing that stands out is August 2012 looks a bit different, one of the
nodes has a higher value compared to previous years. What might have caused
this change? Is this something we can use to split the data into groups? Turns
out in February of 2011, there was an earthquake in Christchurch, and then
another quake in June 2011. The disasters resulted in widespread damage to
buildings, especially older buildings. Come winter 2012, a year of rebuilding
had finished, and people started moving back. The data shows the older
buildings being knocked down, older buildings that relied on chimneys for
heating. With newer buildings, electricity is used to heat the houses, so it
is expected that winter has higher electricity demand compared to summer.

## April 2015

The next weird bleep looks to be a higher than historical average April month.
This is the fall season for New Zealand, but in 2015 winter came early for
Christchurch. It snowed in April of that year, which resulted in everybody
turning their heaters on a month earlier compared to the past. And then the
winter year itself resulted in a higher overall demand, maybe due to it being
a colder winter, or a longer winter.

## Picking the easy winnings

The two signals I listed above are just a starting point, the next step is to
continue diving deeper into the data to figure out what more there is to find.
This is all part of the data analysis fun, being able to take numbers and tell
a story.

Please enable JavaScript to view the [comments powered by Disqus.][3]
[comments powered by Disqus][4]

   [1]: http://billyfung.com

   [2]: /writing/2017/09/chch_quake.png

   [3]: http://disqus.com/?ref_noscript

   [4]: http://disqus.com

[#][1] Aug 31, 2017

# So many options, which one is best?

One of my current projects requires me to use an EC2 for some beefier
computation, but I actually don't know too much about EC2 apart from using the
most basic `t2` instances. Nothing I've worked on has really needed a server
with 60gb of RAM. Until now that is.

## How much memory do you need?

One of the first options that will limit you is memory for your usage. There
are so many different instance types depending on how fast you want your
CPU(s), but largely the difference in price is from storage. You can sacrifice
a couple ghz in speed for a couple GB in RAM, and vice versa.

Working in R, my dataset has 44,000,000 rows and 17 columns. Approximating all
the data to be 10 bytes, this gives us:

[code]

    bytes
    >>> 44000000*17*10
    7480000000
    
[/code]

Which is rounded up to be 7.5GB of data. This sets a baseline of how much RAM
you will need at a minimum for your instance type. You will also need to
account for OS overhead, which will be something like ~.5GB

## What does your program do?

In my case, I'm building a tree based model, which I know will be memory
intensive. Apart from doing that, even setting up the data will require more
memory overhead for manipulation of data. This is why when dealing with large
datasets, using an appropriate memory-efficient data structure is very
important. For R, this means using `data.table`, which will let you perform in
place calculations without having to duplicate your data. Duplicating data
will come at a very high memory cost when you have large amounts of data.

This is where memory estimation gets a bit tricky. Depending on the algorithm
used for your model, this will determine how much memory is required.
Essentially, this is the space complexity of an algorithm, remember all that
stuff you learned from algorithms class but forgot? Yeah that stuff.

Each algorithm will be different, for my case I'm looking at gradient boosted
regression trees (xgboost). The algorithm will have built in measures to
optimise for memory, such as using [histograms][2] to optimise the growing of
the trees based on differences between parent and sibling.

With that said, knowing the baseline requirement, I would suggest running the
script for a baseline version of N trees and M depth, and then scale it
appropriate and approximate up. My parameters for the model are:

[code]

    ntrees = 500
    max_depth = 10
    
[/code]

With this, I know that 7.5GB baseline will probably require more than 30GB RAM
just from thinking about the number of trees built. Of course this has a lot
to do with how the algorithm will work, but it's always better to have too
much RAM than too little.

## CPU requirements

Something that is also important is how fast you need your CPU to be.
Obviously there will be a point in which paying for an hour of a slower CPU
will be the better option if you need to satisfy a memory requirement.

For me, I went with [AWS R4 instance type][3] which is a memory optimised
instance that offers the best value in terms of decent CPU speed and large
amounts of RAM. The `r4.2xlarge` has 61GB of RAM and 8 vCPUs for $0.4 an hour.

That said, I'm still very new to all the offerings of AWS and the different
compute and memory types. I've tried using `c4` instance types in the past and
didn't feel like they were the right choice for my application because of how
little RAM they have.

Please enable JavaScript to view the [comments powered by Disqus.][4]
[comments powered by Disqus][5]

   [1]: http://billyfung.com

   [2]: https://github.com/dmlc/xgboost/issues/1950

   [3]: https://aws.amazon.com/about-aws/whats-new/2016/11/introducing-amazon-
ec2-r4-instances-the-next-generation-of-memory-optimized-instances/

   [4]: http://disqus.com/?ref_noscript

   [5]: http://disqus.com

[#][1] Aug 2, 2017

## NASA MODIS data

So something I recently worked on was looking at NASA satellite images. As is
with the scientific method, I was originally curious about snow cover on land
in a particular area, and wondered what kind of information could be learned
from looking at satellite images. A quick search lead me to the [MODIS][2]
data that NASA collects, and publishes.

With snow cover information, it allows you to form a view onto how much water
inflow there might be in the future. But how does one quantify snow cover?
From the point of view of a satellite, looking down on an area and capturing
an image, you will get:

![Naki][3]

Now how do you determine what is snow cover?

## Normalized Difference Snow Index (NDSI)

Luckily we have this numerical indicator that a smart person came up with,
that gives us a ratio of spectral bands over an area, which in turn gives us a
ratio of how much radiation is reflected in each band. Snow absorbs short wave
infrared, and cloud does not, which is the key way of determining the
difference between snow and cloud. This indicator is mainly used in looking at
glacier movement, being able to accurate track the effects of global warming.
NASA has a [publication][4] on it of course.

Again, NASA has a [great tool][5] that allows you to select your area of
interest, and then it will find corresponding tiles for you. And this is the
beginning of exploring GIS. I am not knowledgeable in GIS by any means, and
there is definitely a lot of room for improvement.

## The data

The format of the files are [HDF][6], which is basically just a binary file
format designed to organise large amounts of data. For the MODIS files, the
data structure is that of raster images, which is a matrix structure of
pixels. To deal with the HDF file formats, I use the [h4toh5][7] converter
tool, to change it into a format that I can use with Python hdf library, which
mainly deal with the h5 format. After all the data is in the proper format for
Python, we are able to begin.

## Looking at the area of interest

![Naki][8]

The first step is to only look at pixels that we care about in each tile. If
you are lucky, then you might only have to look at a single tile that contains
all your information. But sometimes the area you want to look at happens to
need more than a single tile. The first step in grabbing the pixels of
interest is to convert the MODIS projection into longitude and latitude.

I do this with the `pyproj` package:

[code]

    sinu = pyproj.Proj("+proj=sinu +R=6371007.181 +nadgrids=@null +wktext")
    wgs84 = pyproj.Proj("+init=EPSG:4326")
    lon, lat = pyproj.transform(sinu, wgs84, xv, yv)
    
[/code]

If you have a KML file, say from Google Maps or Google Earth, then you can
convert it into a numerical polygon using `shapely` package. Then essentially
what I've done is loop over all the points in a single satellite tile(s)
lon/lat and check if it falls within the shape of interest, and output a 2D
numpy array that indicates whether or not that pixel is one we care about.
This gives us a masking layer to apply over the satellite images

## Extracting pixels of interest

With the mask layer is effectively a filter, letting us extract pixels that we
care about from each satellite image. The task is then to grab the NDSI values
from the tile for the pixel that falls within our area of interest.

The rest of the steps involve doing smoothing and clustering, and then finally
some averaging to give an output NDSI factor for a particular area of
interest.

## Packages used

[code]

    import h5py
    import pyproj
    import keytree
    import numpy as np
    import psycopg2
    from scipy import interpolate
    from shapely.geometry import box, shape
    from astropy.convolution import convolve, Gaussian2DKernel
    from sklearn.cluster import KMeans, AgglomerativeClustering, MiniBatchKMeans
    from sklearn.preprocessing import scale
    
    import matplotlib as mpl
    import matplotlib.pyplot as plt
    from mpl_toolkits.basemap import Basemap
    
[/code]

Some of these will require local distributions, I did this all in Linux, so
`apt-get` got me all the packages I needed.

## Improvements

Is this the best way of solving the task I had? I'm not totally sure, I
learned about [PostGIS][9] and spatial indexing recently, and it seems like a
pretty useful tool, although maybe not needed in my application. The slowest
part of my Python script is when I was extracting and processing pixels from
each tile, so some improvements could be made there.

All in all, it was a fun exercise and it's pretty neat being able to play with
map objects.

Please enable JavaScript to view the [comments powered by Disqus.][10]
[comments powered by Disqus][11]

   [1]: http://billyfung.com

   [2]: https://en.wikipedia.org/wiki/Moderate-
resolution_imaging_spectroradiometer

   [3]: /writing/2017/08/Taranaki.jpg

   [4]: https://modis.gsfc.nasa.gov/sci_team/pubs/abstract.php?id=03224

   [5]: https://search.earthdata.nasa.gov/search

   [6]: https://en.wikipedia.org/wiki/Hierarchical_Data_Format

   [7]: https://ftp.hdfgroup.org/h4toh5/

   [8]: /writing/2017/08/shape.jpg

   [9]: http://postgis.net/

   [10]: http://disqus.com/?ref_noscript

   [11]: http://disqus.com

[#][1] Aug 23, 2017

# Is the Hadleyverse the only option?

One of the major downsides to learning R is figuring out which packages to
use. R comes with many standard packages, but then sometimes they aren't the
most intuitive way to handle your data, or they can be straight up bad. Many
people often direct new learners to use the packages writtein by [Hadley
Wickham][2], which I will say are great packages.

## dplyr

Most of the packages deal with handling data in an intuitive way that makes
your work easier to follow, and easier to read. One such package is `dplyr`,
which is used to manipulate data, mainly data frames.

Example of filtering a dataframe for rows where a column is a specific value.

[code]

    starwars %>%
      filter(species == "Droid")
    
[/code]

The beauty in `dplyr` lies in being able to chain multiple operations
together, so you don't need to create multiple dataframes. In base R, I found
myself creating many `tmp` dataframes that I used to hold data as I was doing
multiple operations on it.

[code]

    starwars %>%
      group_by(species) %>%
      summarise(
        n = n(),
        mass = mean(mass, na.rm = TRUE)
      ) %>%
      filter(n > 1)
    
[/code]

# But is there something faster?

One of the downsides of manipulating dataframes is that they aren't the most
nimble things to be moving around. I've been using `dplyr` and other
Hadleyverse packages for quite awhile now, but lately I've been dealing with
datasets that are much larger, like <5m rows. So the manipulation of data
becomes much slower, and grouping operations start to lag.

## data.table

This is a package that lets a `data.table` inherit from `data.frame`.
Essentially somebody created this package to improve on memory usage, along
with speed. There isn't any piping or elegant usage of writing code that makes
a datatable stand out, because that is a very subjective topic.

What it does do better is that I have found `fread/fwrite` to be way faster at
reading/writing files. Much much faster compared to `read_csv/write_csv`.

With operations that modify the data, data.table can update by reference, this
saves computation and memory cost in assigning the result back to a variable.

[code]

    DT[x > 0, y:= 'positive']
    
[/code]

That simple code will update the data.table y column in-place. As table size
grows, data.table will do that a lot quicker compared to dplyr and data.frame.

The dplyr and data.frame equivalent.

[code]

    new_DF <- DF %>%
        mutate(y = replace(y, which(x > 0), 'positive'))
    
[/code]

So far I am still quite new to data.table, but I have found that I am enjoying
using it because of the speed and memory gains, and I don't find that I wish
it had dplyr like syntax. Aggregating and joining is so much faster with large
tables that I wish I found out about data.table earlier so then I could gain
the intuitive for writing consistent code.

Every bit of optimisation counts when your datasets start growing, so learning
the fastest way first will save you time in the long run.

Please enable JavaScript to view the [comments powered by Disqus.][3]
[comments powered by Disqus][4]

   [1]: http://billyfung.com

   [2]: http://hadley.nz/

   [3]: http://disqus.com/?ref_noscript

   [4]: http://disqus.com

[#][1] Aug 21, 2017

# MS-DOS to bash

One of the programs I've run into is written for Windows only, I could
determine this when I saw all the filepaths.

## Filepaths

Most of the difference lies in changing out a forward slash for a back slash.

[code]

    '%system.fp%..\Input\'
    
[/code]

becomes ` '%system.fp%../Input/' `

## `erase` `move` `copy` `if exist`

These are all MSDOS commands that I had to convert to bash ones.

The corresponding ones are: `rm`, `mv`, `cp`

`if exist` doesn't need a corresponding command, because if if bash doesn't
find anything, then the command just isn't executed.

## `mkdir`

Stays the same!

## The hidden one

One difference I wasn't able to just see immediately was `cls`. Corresponding
command is `clear`

Please enable JavaScript to view the [comments powered by Disqus.][2]
[comments powered by Disqus][3]

   [1]: http://billyfung.com

   [2]: http://disqus.com/?ref_noscript

   [3]: http://disqus.com

[#][1] Aug 30, 2017

## It began in 2016

Recently I had a coworker ask me "Why did you guys choose to use Docker?" (and
various other tools). To which I didn't really have a reply except that "it
was hyped up and popular at the time". Which really was the case for my first
usage of Docker. Back in 2016, Docker hype was building and lots of people
were exploring into using it.

So we hopped abroad the train for one of our newer and smaller scoped
projects. The regular stack used Vagrant and Virtualbox and virtual machines,
the regular battle tested stack. But Docker offered flexibility and ease of
use, the phrase "disposable VM" comes to mind. But in 2016 the toolchain for
Docker and OSX wasn't full matured yet. There were still many bugs and the
experience itself of setting up and being able to do what we wanted was
difficult.

### Documentation

Documentation is probably one of the most feared tasks for any large software
project. Especially one that becomes popular and people start criticising it.
I should not be one to talk about documentation, but using Docker in 2016
largely lead to issues because documentation wasn't adequate, and the
community wasn't mature enough yet. More often than not, stackoverflow became
what we used to get answer to problems, which isn't ideal.

## Docker in 2017

Now in 2017, the Docker toolchain has matured a lot, no more fiddling around
with `docker-machine` and playing around with networking. Although all the
time spent figuring out networking was valuable knowledge, the networking
between containers has been much improved. The OSX docker app works
beautifully, and there are no more issues with memory leaks or crashing.

I wish I documented more my issues with Docker from a year ago, which is why
I'm going to try to write more about my experiences with Docker in 2017.

### Disposable VM

Currently I'm using Docker to set up images where I'm slowly building up
various tools that work with [GAMS][2]. Docker is useful for this because I
can write a Dockerfile that is based off a Linux OS, Ubuntu in my case, and
then have it download GAMS, and setup GAMS in Linux.

Then after having GAMS set up, I saved the image and pushed it to [Docker
hub][3] to share with the community. This allows anybody to pull the GAMS
Linux image and use it for whatever they want. This also makes it a lot easier
for me to slowly add more software tools to the image, and then build it up
and share again. As it is, the image is 2GB, which seems fairly large, but
compared to setting up a Windows OS and installing it, it's not much at all.

## What's next

Next will be to try out Docker swarm and set up a multiple containers that
talk to each other, and see how easy it is to manage. Although these days I
hear [Kubernetes][4] is all the hype.

Please enable JavaScript to view the [comments powered by Disqus.][5]
[comments powered by Disqus][6]

   [1]: http://billyfung.com

   [2]: https://www.gams.com/

   [3]: https://hub.docker.com/r/bfung/gams/tags/

   [4]: https://kubernetes.io/

   [5]: http://disqus.com/?ref_noscript

   [6]: http://disqus.com

[#][1] Aug 17, 2017

# You're never too smart for dumb questions

One of the main benefits of pair programming is the fact that you'll probably
encounter questions you've never thought of people. Especially if you pair
with somebody that doesn't have much knowledge of the programming language and
you are trying to teach them. It's refreshing to get questions from an
inquiring mind that has just been recently exposed to something completely
new.

I don't consider myself an expert Python programmer, especially since I always
get asked questions I can't answer. I know how the language works, and it's
features, but I still learn a lot from getting asked very specific questions
that I normally don't think of. Usually the questions pertain to things within
the programming language that I gloss over, or haven't fully learned.

Being asked questions also forces you to reinforce your own understanding. I
try my best to think of analogies as answers if possible, trying to connect
something abstract to something more tangible. This is how I learned most of
everything I was taugh in uni, taking abstract concepts and thinking of them
in a way that makes them concrete.

For example, I was asked yesterday about Docker, and how Docker works within
our development environment. Something that is difficult to do is finding the
right balance between too much information, and too little information. To a
beginner, eyes get glossy when you start delving into very technical jargon,
so you have to find the right balance. I explained that Docker is like an Ikea
instruction manual, very simple and no words; no matter what language you
speak you can build the same final product.

That might've been a terrible analogy, but I never really thought about what
Docker was before, only what it could do and how we could use it.

Moral of the story: Seek out mentoring/pairing opportunities because you will
learn more, and it's good fun.

Please enable JavaScript to view the [comments powered by Disqus.][2]
[comments powered by Disqus][3]

   [1]: http://billyfung.com

   [2]: http://disqus.com/?ref_noscript

   [3]: http://disqus.com

[#][1] Aug 24, 2017

# Differences between Flask and Django - Part 1

This blog post is highly biased because I have worked a lot with Flask, and
not at all with Django. So coming from a Flask background where I don't always
follow a rigid MVC framework, I'll write about what I discover when comparing
the two.

The list of tasks comes from me working on a website to host podcasts that my
[friend][2] and I record.

# 1\. Rendering a static page

So I have a webpage where I want to display an "About Us". This will be static
information and won't be require any models (well I won't say no to a model…)
Simple, easy task to learn the ins and outs of the framework.

## Flask:

  * app.py
  * templates/about.html

You could have the html all within the route return as well if you wanted to
be gross about it.

[code]

    from flask import Flask, render_template
    app = Flask(__name__)
    
    @app.route('/about')
    def about():
        return render_template('template/about.html')
    
[/code]

## Django

  * templates/about.html
  * urls.py
  * views.py

Since Django comes with "batteries included", when you start the project,
everything you need is already there. You just have to know which pieces to
poke around in.

The `urls.py` file is where the routing is done within Django, similar to the
route decorator in Flask. The routing is a bit more powerful in Django though,
with the `django.conf.urls` utility functions.

The normal usage is to define a list of `url()` instances that take on the
following parameters:

[code]

    url(regex, view, kwargs=None, name=None)
    urlpatterns = [
        url(r'^index/$', views.index, name='main-view'),
        url(r'^about/',  views.about),
        ...
    ]
    
[/code]

The regex usage in Django comes built in, and is quite powerful although I
personally have never thought about such a utility until I saw it in Django.
With Flask, using `werkzeug.routing` you can build your own [regex converter
class][3] that inherits from `BaseConverter`

The `views.py` change is just so that the Django project is already well
organised and data is abstracted through it's proper layers. All it's doing is
taking the request for the `/about` page and then returning the render of
`about.html`

### Summary

I know this was a very simple task, but it helps to explain how Django
operates, and how different/flexible Flask can be for certain things. If I
wanted to make a web application that only served static pages, then it would
seem like Django has so much extra overhead that isn't needed. It's like
buying a formula 1 car to drive 1km to work.

Please enable JavaScript to view the [comments powered by Disqus.][4]
[comments powered by Disqus][5]

   [1]: http://billyfung.com

   [2]: https://jinpark.net

   [3]: http://werkzeug.pocoo.org/docs/0.12/routing/#custom-converters

   [4]: http://disqus.com/?ref_noscript

   [5]: http://disqus.com

[#][1] Jun 30, 2017

## You don't know till you know

I find that whenever I'm in a rush to get something working, I have a bad
habit of just getting into writing code and getting the thing done. More often
than not, this results in messy code, and/or slow code. And then when I need
to do something similar again, I just re-use the original code and end up with
a potential compounded problem.

One of the more common tasks that I repeatedly do is take a bunch of files and
insert the data into a Postgres database. Up until recently, I haven't had to
deal with large enough datasets, so my poorly written code was still
acceptable in terms of execution time. Usually this means executing the insert
script locally on a test database first, and then on production. I'm using
Python 3 and the [Psycopg2][2] postgres driver.

## Psycopg2 execute and execute_values

The original code looked something like:

[code]

    def insert_data(filename, date):
    
        sql = """
        INSERT INTO test (a, b, c, d)
        VALUES (%s, %s, %s, %s)
        """
    
        with open(filename) as csvfile, get_cursor() as c:
            reader = csv.reader(csvfile)
            header = next(reader)
    
            for row in reader:
                n = row[0][2:]
                values = (row[1], date, n, row[2])
                c.execute(sql, values)
    
[/code]

This looks like a pretty innocent looking insert function, it takes the file
loops over every row and inserts it into the table.

The refactored function looks like:

[code]

    def insert_data(filename, date):
    
        sql = """
        INSERT INTO test (a, b, c, d)
        VALUES %s
        """
    
        with open(filename) as csvfile, get_cursor() as c:
            reader = csv.reader(csvfile)
            header = next(reader)
            values_list = []
    
            for row in reader:
                n = row[0][2:]
                values = (row[1], date, n, row[2])
                values_list.append(values)
    
            execute_values(c, sql, values_list)
    
[/code]

The difference between these two functions is the `execute` and
`execute_values`. Each time you use `execute`, psycopg2 does a complete return
trip from the database to your computer, so this means it will execute the row
INSERT to the database server, and then return. A functionality with Postgres
is that you can insert multiple rows at a time, and this is what
[`execute_values`][3] does.

Instead of inserting a single row query, the refactored version creates a
query with multiple rows to insert. This reduces the number of round trips to
the database server drastically, and results in much faster performance.

## Run times

I ran the two different functions on a small subset of data being 288478 rows,
which happens to be 3% of the files I was inserting.

_`execute`_ code:

[code]

    real    0m54.761s
    user    0m12.752s
    sys     0m4.876s
    
[/code]

_`execute_values`_ code:

[code]

    real    0m7.545s
    user    0m2.092s
    sys     0m0.544s
    
[/code]

Well that's a 700% increase on just a small number of rows. I didn't bother to
compare what the times would be like for the complete dataset, but running the
refactored version took about 25 minutes, so the original version would've
taken hours!

## Lesson learned

Would I have saved time if I had looked more in depth at the docs before
writing my code? Most likely. I think lessons like this is what separates
senior and junior level programmers, those who are able to grasp the scope of
a problem and it's solutions before they even begin writing code. Meanwhile,
the juniors have to take time to write bad code in order to learn.

## Update, another test

So my friend pointed me to another Python package called [dataset][4] saying
it's what he uses because he's a lazy Python user who is allergic to SQL.
[He][5] also said that Python > SQL, so I decided to prove him wrong, and also
because I didn't believe that another package that uses SQLAlchemy would be
faster than just using Psycopg2. (SQLAlchemy is built on Psycopg2)

[code]

    db = dataset.connect('postgresql://test@127.0.0.1:5432/testdb')
    def insert_demand_data(filename, date):
        with open(filename) as csvfile:
            reader = csv.reader(csvfile)
            header = next(reader)
            values_list = []
            for row in reader:
                n = row[0][2:]
                values = dict(node=row[1], date=date, n=n, r=row[2])
                values_list.append(values)
            table.insert_many(values_list)
    
[/code]

And what do you know.

[code]

    real    0m53.392s
    user    0m17.512s
    sys     0m3.112s
    
    testdb=# select count(*) from test ;
     count
    --------
     288478
    (1 row)
    
    
[/code]

Please enable JavaScript to view the [comments powered by Disqus.][6]
[comments powered by Disqus][7]

   [1]: http://billyfung.com

   [2]: http://initd.org/psycopg/

   [3]: http://initd.org/psycopg/docs/extras.html#fast-exec

   [4]: https://dataset.readthedocs.io/en/latest/index.html

   [5]: https://www.jinpark.net/

   [6]: http://disqus.com/?ref_noscript

   [7]: http://disqus.com

[#][1] Jun 19, 2017

## Making a win-win situation

Whenever you make a trade, you enter into it with some risk, risk of losing or
gaining money. Hedging is essentially making sure you keep that risk as low as
possible, so maybe when you win you don't win as much, but you won't also lose
as much. The goal is to try to keep risk as low as possible, while maximising
gain.

Within most markets, there will be products that are correlated, meaning the
pair of prices move together, be it negatively correlated where when one goes
up, the other goes down. Or positively correlated where they both go up
together. By finding these pairs, you can effectively form a view on where the
price will go, and then make trades accordingly.

## Searching for the one(s)

Most of the difficulty in hedging lies in understanding the market, and
finding which products will actually move according to your theory. While I
have no expertise within financial markets, with electricity markets the fact
that there is a physical grid provides insight into where to start looking.
Nodal pricing markets have network losses and constraints, and often nodal
markets will have products that help hedge against this risk. Long story
short, these products, called FTRs provide a vehicle for hedging your risk in
the futures market.

When you have 1 FTR, and 1 Future, you hold them for the purpose of being able
to manage your risk if the spot price happens to rise, you will be covered in
the FTR. To make a trade, you want to form a view on where the market will
move, if you know that the futures will trade at price X, and the FTRs will
clear at Y price, then you can ask the question of what you want to sell and
buy.

## Factors/Features

When dealing with the grid, you want to look for certain behaviors that might
enhance a hedge pair relationship. Often looking at historical data, you might
be able to spot large changes to the market that might separate the data into
different regimes.

![Site][2]

Sometimes you have to make a very large plot to see the behaviors change over
time, usually it's bad practice to do this but I couldn't think of a better
way to visualise it. But on a large screen, some changes are more obvious than
others, and it let's you prune down the dataset to certain regimes.

The general process is to look at the dataset as a whole, use your findings to
subset the data into smaller datasets.

Please enable JavaScript to view the [comments powered by Disqus.][3]
[comments powered by Disqus][4]

   [1]: http://billyfung.com

   [2]: /writing/2017/06/blobs.png

   [3]: http://disqus.com/?ref_noscript

   [4]: http://disqus.com

[#][1] Jun 14, 2017

## Making something from nothing

More often than not, I deal with datasets where I don't have a multitude of
features to choose from. It's nothing like the Kaggle competitions where you
usually have too many features, and need to figure out how to cull down the
list. Sometimes I wonder what it would be like to work in an environment where
you have people working on building a feature complete dataset, and the
analysis team gets handed something where the data is as complete as possible.

Something I've been dealing with recent is a dataset where there are only 3
columns of data: `the date, numeric_A, numeric_B`. The plot of A vs B looks
something like:

![Site][2]

So essentially I'm given time series data, with data that looks like it could
be linearly correlated. But how do we figure out more features, to even being
to explore?

### Seasonality

Of course this really depends on the data you're dealing with, but sometimes
you can extract a feature of what season the date is. This can simply be
looking at the month and estimating what the season it is, or you can put in
specific dates for each year and then bin them more accurately.

[code]

    getSeason <- function(dates) {
        d <- month(dates)
    
        ifelse (d >= 12 | d < 3, "Summer",
          ifelse (d >= 9 & d < 12, "Spring",
            ifelse (d >= 3 & d < 6, "Fall", "Winter")))
    }
    
[/code]

Is there any other nature of the data that you can quantise? Basically the
overview is to look at the year, month, and day components of the date and see
if you can sort them into bins. Then accordingly you should have a couple more
features that you can work with, and maybe combine to find some more.

Please enable JavaScript to view the [comments powered by Disqus.][3]
[comments powered by Disqus][4]

   [1]: http://billyfung.com

   [2]: /writing/2017/06/Rplot.png

   [3]: http://disqus.com/?ref_noscript

   [4]: http://disqus.com

[#][1] Jun 29, 2017

## Sometimes you have to use proprietary software

R and Python are both tools I use for exploring data, and building
mathematical models. More often than not, they are all you really need to use
and probably all you will ever encounter. But when you start dealing with
linear systems, where there isn't a single solution to a problem, but multiple
solutions that differ in minute amounts, you will need to start optimising for
an objective function.

Linear programming looks something like having multiple equations of lines as
constraints, and then adjusting the lines such that they cross at an optimal
point.

![Site][2]

The common textbook example is optimising supply and demand. You need to
transport products from one location to another, so you have multiple
products, multiple locations. The optimisation will be to transport as many
products as possible, to maximise the profit. Each product is located in a
location, with different costs to ship to different locations. Then you can
start to build the model of linear equations that constraint the system.

With these type of problems, you end up with a system of linear equations that
constrain the output. Luckily enough, very smart people have written solvers
that are built for certain types of systems specifically. So if you have a
linear programming problem that fits into a certain criteria, chances are
there is a solver that somebody has written already for it. The problem is
that it is usually not free.

## Hardness

After reading all that, linear programming doesn't really seem that difficult
to solve right? Linear programming is P hard in complexity, so this means that
the domain of the problem can be approached with algorithms that allow linear
programming to solved more easily than others. To make things a bit more
complex, another type of optimisation problem involves solving for Integer
Programming, or Mixed Integer Programming adds another level of complexity.
Integer Programming restricts all the variables to be integers, and results in
a NP-hard problem. With LP, polynomial time is achievable. And with MIP, being
NP-hard means there is no best case, only worst case exponential.

This means another couple PhD papers were written for proprietary solvers.

## Electricity Grid solving

All this has to do with my current work because I'm working on using GAMS, a
programming language that is written for building algebraic models. A lot of
solvers are written so that they can interface with GAMS, and GAMS itself is a
language that is based off set theory, so it is very mathematical in nature to
begin with. This makes it easier to build the scaffolding for your
mathematical model.

GAMS projects are usually structured so that you have you input data, your
variables, your objective functions, and then you tell it to solve.

An electricity grid is essentially managed by a very large number of
constraints. You have transmission constraints, like outages between certain
nodes. You have generation constraints, where there are a number of power
stations and only so much power that can be provided. You have distribution
constraints, which involve which the fact that only certain distributors exist
in certain areas. All this is a very generalised overview of what goes into
the model before it is solved.

Formulation of the problem requires consideration of the available solvers,
and of course the available time. It can be possible that you create a model
that will take too long to converge to a desireable outcome. For a power grid,
this model formulation usually comes down to being Mixed Integer Programming.

## Choosing a solver

So with GAMS already being proprietary software, I was out of my element. I
learned further that the best solvers are licensed. There are solvers included
within GAMS, so that's what I started off with. But I've learned that IBM's
CPLEX solver is the defacto industry standard for MIP problems, so look for a
review/comparison in the future.

Please enable JavaScript to view the [comments powered by Disqus.][3]
[comments powered by Disqus][4]

   [1]: http://billyfung.com

   [2]: /writing/2017/06/linear_p.gif

   [3]: http://disqus.com/?ref_noscript

   [4]: http://disqus.com

