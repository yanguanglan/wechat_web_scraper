## Example Usage && Demo

- Scraping. I scrape all articles from wechat page(see below) using selenium:
![collections](http://ww3.sinaimg.cn/large/72f96cbagw1f5jos85vunj21kw0sx7bo.jpg)

- Training. `curl -X GET -H "X-API-TOKEN: <api-token>" -H "Content-Type: application/json; charset=utf-8" http://127.0.0.1:5000/train -d "{\"data-url\": \"ada-content-en.csv\"}"`. Only using a correct api-token can trigger the machine training(with the below image messege shown).
![training finished](http://ww4.sinaimg.cn/large/72f96cbagw1f5jowfjtuyj20eo00y3yl.jpg)

- Predicting. `curl -X POST -H "X-API-TOKEN: <api-token>" -H "Content-Type: application/json; charset=utf-8" http://127.0.0.1:5000/predict -d "{\"item\":1,\"num\":10,\"data-url\": \"ada-content-en.csv\"}"`. Will receive something like:
![predicting finished](http://ww1.sinaimg.cn/large/72f96cbagw1f5jozydqf6j20ym0t4aji.jpg)


## Dependency

### scraper

Install selenium, run `pip install selenium`, and chromeDriver.

### content engine

Install python library dependency in virtual environment using "conda", and redis by `brew install redis`.

## Logs

### scraper

Check [this post](http://docs.python-guide.org/en/latest/scenarios/scrape/) for basic python web scraping, but is is only fetching the static html page, without rending javascript part of code, since some content of the page is generated by javascript, we want to simulate a browser environment to get an complete html page for scraping.(selenium?)

From [this post](http://stackoverflow.com/questions/29449982/installing-pyqt4-with-brew), we can install `PyQt4` by doing `brew install PyQt4 --with-python27`.

The next question would be how to simulate the "load more" event? Only sliding the page to the end can trigger the load more action.

- One option is to use "selenium" to simulate the infinite scrolling event. Since we will be using chromedriver, use `brew install chromedriver` to install. One of the handy thing about selenium is that it allows us to execute javascript script on the selected elements. Check [this post](http://stackoverflow.com/questions/21006940/how-to-load-all-entries-in-an-infinite-scroll-at-once-to-parse-the-html-in-pytho) for more information of how to simulate infinite scroll.(Thus, basically with selenium we don't have to use pyqt4 webkit.)

- The other one is to simple inspect the difference url being used when scrolling happens, find the pattern. But in the wechat website exmple, this trick doesn't work.

After scraping all required static html content as string, we can do regex `findall` to find all matching urls from those three panels. Then save it to csv file. Then hand it to next step of fetching.

With all urls in place, we use beautifulsoup to extract all nested texts from article into an array, then use `filter()` to filter out elements whose parent are styles, scripts and so on. *BeautifulSoup really comes in handy for scraping all texts.* Check [this post](http://stackoverflow.com/questions/1936466/beautifulsoup-grab-visible-webpage-text) for more information. One problem encountered is the encoding, I have a "ascii not recognize" error, use `.encode('utf-8')\.decode('utf-8')` to work around with that. Remember that, *when we type weird characters like Chiese characters or Abraic characters, we use utf-8,* while we better deal with or string in unicode, the procedure of from normal string to unicode is called "decode", while the way to produce string back is called "encode". Check [this post](http://stackoverflow.com/questions/5096776/unicode-decodeutf-8-ignore-raising-unicodeencodeerror) for more illustration.


### content engine

List all python dependencies in `conda.txt`, then run `conda create -n <virtual env's name> --file conda.txt` to create a new environment based on the library from `conda.txt`. Then the following things will be just get into that environment and get out by using: `source activate <env's name>` and `source deactivate`. Check [this post](https://uoa-eresearch.github.io/eresearch-cookbook/recipe/2014/11/20/conda/) for more information about how to remove an environment, note that with conda, we can change the python version when we create conda virtual environment. Use `conda info -e` to list all current environments. 

Then, need to install redis on mac to be tested in local environment(`brew install redis`). Use `ps aux | grep redis` to check if redis server is running. If it's not running, use the following command `nohup redis-server &` to start a redis-server process and let it run in background. Check [this post](http://jasdeep.ca/2012/05/installing-redis-on-mac-os-x/) for more information about install and run redis server.

### web client

Next step, build a simple web client that allows users to upload a chunk of texts and submit, then I translate that texts into English and store in one public.csv file, then merge with ada-content-en.csv and do the TF-IDF again. Then give back a list of similar post. Web node app will spawn a child process that translate texts and construct http calls to engine server, which re-train and predict with new-coming text. Check [this post](http://www.sohamkamani.com/blog/2015/08/21/python-nodejs-comm/) for more information about how to communicate between python and node(with child_process package).

## TODOs

✔️(2016.7.4)After collecting all post url into csv file, we trace up to its pointing article page and scrape for the first three paragraphs(in case of not choking redis for too much content?), then use google translate to make it in English and do TF-IDF training. 

✔️And for now(16.7.4), /selenium/ada.csv contains urls <s>that are repetitive</s> and <s>"wrong"(contains "&amp;" symbols e.t.c)</s>, need later update.

✔️(2016.7.6)Build a simple web client with node.js and create a sub process that runs with python script that translates it into english.

✔️(2016.7.12)Host it on AWS, airloft.org

✔️(2016.7.16)Add a play mode. Allow user to try their own texts with our database, just not updated database. Add an auth like "author has to be ada" e.t.c.

✔️(2016.7.17)Add field checking for forms, so that we allows users to upload empty field when in "playmode".

✔️(2016.7.17)Allow short english text being uploaded and texts, prepared for twitter and facebook feeds.

- A fancier UI?

- Extract and summerize a tag or short description from uploaded text. Then that's basically finding the highest weights from all sentences based on TF-IDF.

- Allow tag2article, pharse2article matching, is it possible by just TF-IDF? or need other algorithm?

## Side Notes

### what is conda?

Conda is a package manager application that quickly installs, runs, and updates packages and their dependencies. The conda command is the primary interface for managing installations of various packages. It can query and search the package index and current installation, create new environments, and install and update packages into existing conda environments. See our Using conda section for more information.

Conda is also an environment manager application. A conda environment is a directory that contains a specific collection of conda packages that you have installed. For example, you may have one environment with NumPy 1.7 and its dependencies, and another environment with NumPy 1.6 for legacy testing. If you change one environment, your other environments are not affected. You can easily activate or deactivate (switch between) these environments. You can also share your environment with someone by giving them a copy of your environment.yaml file. Check [this post](http://conda.pydata.org/docs/intro.html) for installation and more information.

### TextBlob as NL translation

Use TextBlob as a wrapper API for Google translation. Simple `pip install -U textblob` for installation. Try "Goslate" library, but it imposes an API query limit, which causes some inconvienience. Check [this post](https://pypi.python.org/pypi/textblob) for more information.

### Read & Write files with .csv

Check [this post](https://docs.python.org/2/library/csv.html) for more information.

### Manage process on EC2

Using pm2 to manage node process, `pm2 start server.js --name="wechat"` to create a pm2 process, and `pm2 list` can list all live running process. With `pm2 delete <process name>\<process id>` to delete registered process.

Using screen to manage python process, `screen -S wechat` to create a new session in sshing, then run the python web server with `python web.py`, notice that in the case of my EC2, I have all python dependencies installed properly, then I don't have to activate my conda environment with `source activate <env>`, directly running python script should be fine. (using `conda install <package>` could be handy or messy depending on cases.). Then do "CTRL + A, D" to escape from that session, use `screen -ls` to list all live sessions on that time. When resuming to detached session, use `screen -r <session name>\<session id>`. 

One more thing to be noticed, redis server should be alive when this program is running, check your port 6379, it should be taken up by default by redis server.

## Trouble Shooting

### backup.csv format

After backup.csv is complete, avoid directly editing backup.csv file, any direct deleting might potentially change backup.csv format, use `git reset --hard HEAD` to rollback to previous version, do notice that such operation will discard all file changes to the previous committed version. If we want to simply discard changes on one single file, then just do `git checkout <../content\ engine/backup.csv>`.

