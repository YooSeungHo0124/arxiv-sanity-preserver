
안녕하세요!
기본적인 README.md는 하단에 적혀있는 제가 fork했던 내용입니다.

다만 기존의 코드로는 몇가지 동작하는데 에러가 있어 제가 python 코드 몇가지 수정했습니다.
그리고 이것은 제가 가상환경에서 사용한 패키지들의 버전입니다.
참고 부탁드려요.
| Package            | Version    |
|--------------------|-----------|
| blinker            | 1.9.0     |
| certifi            | 2025.8.3  |
| charset-normalizer | 3.4.3     |
| click              | 8.3.0     |
| colorama           | 0.4.6     |
| Deprecated         | 1.2.18    |
| dnspython          | 2.8.0     |
| feedparser         | 6.0.12    |
| Flask              | 2.1.3     |
| Flask-Limiter      | 1.5       |
| future             | 1.0.0     |
| idna               | 3.10      |
| itsdangerous       | 2.2.0     |
| Jinja2             | 3.1.6     |
| joblib             | 1.5.2     |
| limits             | 5.6.0     |
| markdown-it-py     | 4.0.0     |
| MarkupSafe         | 3.0.3     |
| mdurl              | 0.1.2     |
| numpy              | 2.3.3     |
| oauthlib           | 3.3.1     |
| ordered-set        | 4.1.0     |
| packaging          | 25.0      |
| pip                | 25.2      |
| Pygments           | 2.19.2    |
| pymongo            | 4.15.2    |
| python-dateutil    | 2.9.0.post0|
| python-twitter     | 3.5       |
| pytz               | 2025.2    |
| requests           | 2.32.5    |
| requests-oauthlib  | 2.0.0     |
| rich               | 14.1.0    |
| scikit-learn       | 1.7.2     |
| scipy              | 1.16.2    |
| setuptools         | 80.9.0    |
| sgmllib3k          | 1.0.0     |
| six                | 1.17.0    |
| threadpoolctl      | 3.6.0     |
| tornado            | 6.5.2     |
| typing_extensions  | 4.15.0    |
| urllib3            | 2.5.0     |
| Werkzeug           | 2.0.3     |
| wrapt              | 1.17.3    |









# arxiv sanity preserver

**Update Nov 27, 2021**: you may wish to look at my from-scratch re-write of arxiv-sanity: [arxiv-sanity-lite](https://github.com/karpathy/arxiv-sanity-lite). It is a smaller version of arxiv-sanity that focuses on the core value proposition, is significantly less likely to ever go down, scales better, and has a few additional goodies such as multiple possible tags per account, regular emails of new papers of interest, etc. It is also running live on [arxiv-sanity-lite.com](https://arxiv-sanity-lite.com).

This project is a web interface that attempts to tame the overwhelming flood of papers on Arxiv. It allows researchers to keep track of recent papers, search for papers, sort papers by similarity to any paper, see recent popular papers, to add papers to a personal library, and to get personalized recommendations of (new or old) Arxiv papers. This code is currently running live at [www.arxiv-sanity.com/](http://www.arxiv-sanity.com/), where it's serving 25,000+ Arxiv papers from Machine Learning (cs.[CV|AI|CL|LG|NE]/stat.ML) over the last ~3 years. With this code base you could replicate the website to any of your favorite subsets of Arxiv by simply changing the categories in `fetch_papers.py`.

![user interface](https://raw.github.com/karpathy/arxiv-sanity-preserver/master/ui.jpeg)

### Code layout

There are two large parts of the code:

**Indexing code**. Uses Arxiv API to download the most recent papers in any categories you like, and then downloads all papers, extracts all text, creates tfidf vectors based on the content of each paper. This code is therefore concerned with the backend scraping and computation: building up a database of arxiv papers, calculating content vectors, creating thumbnails, computing SVMs for people, etc.

**User interface**. Then there is a web server (based on Flask/Tornado/sqlite) that allows searching through the database and filtering papers by similarity, etc.

### Dependencies

Several: You will need numpy, feedparser (to process xml files), scikit learn (for tfidf vectorizer, training of SVM), flask (for serving the results), flask_limiter, and tornado (if you want to run the flask server in production). Also dateutil, and scipy. And sqlite3 for database (accounts, library support, etc.). Most of these are easy to get through `pip`, e.g.:

```bash
$ virtualenv env                # optional: use virtualenv
$ source env/bin/activate       # optional: use virtualenv
$ pip install -r requirements.txt
```

You will also need [ImageMagick](http://www.imagemagick.org/script/index.php) and [pdftotext](https://poppler.freedesktop.org/), which you can install on Ubuntu as `sudo apt-get install imagemagick poppler-utils`. Bleh, that's a lot of dependencies isn't it.

### Processing pipeline

The processing pipeline requires you to run a series of scripts, and at this stage I really encourage you to manually inspect each script, as they may contain various inline settings you might want to change. In order, the processing pipeline is:

1. Run `fetch_papers.py` to query arxiv API and create a file `db.p` that contains all information for each paper. This script is where you would modify the **query**, indicating which parts of arxiv you'd like to use. Note that if you're trying to pull too many papers arxiv will start to rate limit you. You may have to run the script multiple times, and I recommend using the arg `--start-index` to restart where you left off when you were last interrupted by arxiv.
2. Run `download_pdfs.py`, which iterates over all papers in parsed pickle and downloads the papers into folder `pdf`
3. Run `parse_pdf_to_text.py` to export all text from pdfs to files in `txt`
4. Run `thumb_pdf.py` to export thumbnails of all pdfs to `thumb`
5. Run `analyze.py` to compute tfidf vectors for all documents based on bigrams. Saves a `tfidf.p`, `tfidf_meta.p` and `sim_dict.p` pickle files.
6. Run `buildsvm.py` to train SVMs for all users (if any), exports a pickle `user_sim.p`
7. Run `make_cache.py` for various preprocessing so that server starts faster (and make sure to run `sqlite3 as.db < schema.sql` if this is the very first time ever you're starting arxiv-sanity, which initializes an empty database).
8. Start the mongodb daemon in the background. Mongodb can be installed by following the instructions here - https://docs.mongodb.com/tutorials/install-mongodb-on-ubuntu/.
  * Start the mongodb server with - `sudo service mongod start`.
  * Verify if the server is running in the background : The last line of /var/log/mongodb/mongod.log file must be - 
`[initandlisten] waiting for connections on port <port> `
9. Run the flask server with `serve.py`. Visit localhost:5000 and enjoy sane viewing of papers!

Optionally you can also run the `twitter_daemon.py` in a screen session, which uses your Twitter API credentials (stored in `twitter.txt`) to query Twitter periodically looking for mentions of papers in the database, and writes the results to the pickle file `twitter.p`.

I have a simple shell script that runs these commands one by one, and every day I run this script to fetch new papers, incorporate them into the database, and recompute all tfidf vectors/classifiers. More details on this process below.

**protip: numpy/BLAS**: The script `analyze.py` does quite a lot of heavy lifting with numpy. I recommend that you carefully set up your numpy to use BLAS (e.g. OpenBLAS), otherwise the computations will take a long time. With ~25,000 papers and ~5000 users the script runs in several hours on my current machine with a BLAS-linked numpy.

### Running online

If you'd like to run the flask server online (e.g. AWS) run it as `python serve.py --prod`.

You also want to create a `secret_key.txt` file and fill it with random text (see top of `serve.py`).

### Current workflow

Running the site live is not currently set up for a fully automatic plug and play operation. Instead it's a bit of a manual process and I thought I should document how I'm keeping this code alive right now. I have a script that performs the following update early morning after arxiv papers come out (~midnight PST):

```bash
python fetch_papers.py
python download_pdfs.py
python parse_pdf_to_text.py
python thumb_pdf.py
python analyze.py
python buildsvm.py
python make_cache.py
```

I run the server in a screen session, so `screen -S serve` to create it (or `-r` to reattach to it) and run:

```bash
python serve.py --prod --port 80
```

The server will load the new files and begin hosting the site. Note that on some systems you can't use port 80 without `sudo`. Your two options are to use `iptables` to reroute ports or you can use [setcap](http://stackoverflow.com/questions/413807/is-there-a-way-for-non-root-processes-to-bind-to-privileged-ports-1024-on-l) to elavate the permissions of your `python` interpreter that runs `serve.py`. In this case I'd recommend careful permissions and maybe virtualenv, etc.
