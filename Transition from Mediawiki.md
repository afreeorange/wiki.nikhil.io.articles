## Resources

Used `python-markdown` as the converter.

* [Nice extension bundle](http://facelessuser.github.io/pymdown-extensions/). Used all of it.
* [List of 3^rd^ Party Extensions for `python-markdown`](https://github.com/waylan/Python-Markdown/wiki/Third-Party-Extensions)
* [Base16](https://chriskempson.github.io/base16/#default) [Pygments CSS](https://github.com/idleberg/base16-pygments) for code-highlighting
* [A discussion](http://lepture.com/en/2014/markdown-parsers-in-python) of popular Python Markdown parsers.
* [Basscss](http://www.basscss.com/docs/base-reset/) looks interesting

## Exporting MediaWiki Content

Use [dumpBackup.php](https://git.wikimedia.org/blob/mediawiki%2Fcore.git/HEAD/maintenance%2FdumpBackup.php)

* in the `maintenance` folder
* with a `--full` flag

to get all pages and revisions. This dumps an XML file. It needed to be parsed.

Using Python 2.7 since `gittle` has a [`urlparse`-related bug for Python 3](https://github.com/FriendCode/gittle/issues/49).

## Parsing XML and Making a `git` repo

```python
# -*- coding: utf-8 -*-

from __future__ import print_function
import shlex
import sys
import subprocess
from multiprocessing import Pool, cpu_count

import sh
from gittle import Gittle
from bs4 import BeautifulSoup

# UTF-8...
reload(sys)
sys.setdefaultencoding('UTF8')

output_folder = '/Users/nikhil/Desktop/wiki/articles'
tmp_folder = '/Users/nikhil/Desktop/wiki/tmp'
path_to_xml_dump = '/Users/nikhil/Desktop/wiki/wiki.xml'
committer_name = 'Nikhil Anand'
committer_email = 'mail@nikhil.io'

process_pool = Pool(cpu_count() * 2)

# Clean all folders
sh.rm('-rf', output_folder, tmp_folder)
print('Removed folders')

# Create required folders
sh.mkdir(output_folder, tmp_folder)
print('Created folders')

# Initialize git repo
repo = Gittle.init(output_folder)
print('Initialized git repo')
repo.commit(name=committer_name, email=committer_email, message='Initial commit')

# Get all pages from XML dump
dump = BeautifulSoup(open(path_to_xml_dump), "lxml").findAll('page')
print('Found', len(dump), 'pages')

page_counter = 0

for page_node in dump:

    page_title = page_node.title.text.replace('/', ' or ')
    revisions = page_node.findAll('revision')
    revision_counter = 1

    for revision in revisions:

        wikifile = '{}.{}.mediawiki'.format(
                        page_title,
                        revision.timestamp.text
                    )

        markdownfile = '{}.md'.format(page_title)

        # Get the content for the current revision
        revision_text = revision.find('text')

        # Write to the temp file
        with open('{}/{}'.format(tmp_folder, wikifile), 'w') as f:
            f.write(revision_text.text)
            print('Wrote {}'.format(wikifile))

        # Use Pandoc to convert file
        command = 'pandoc +RTS -K256m -RTS -f mediawiki -t markdown_mmd "{}/{}" -o "{}/{}"'.format(
                        tmp_folder,
                        wikifile,
                        output_folder,
                        markdownfile
                    )

        conversion_process = subprocess.Popen(shlex.split(command))
        conversion_process.wait()
        print('Converted {}'.format(wikifile))

        # Stage the file
        repo.stage(markdownfile)

        # Commit
        if revision_counter == 1:
            commit_message = '{} : First Draft'.format(page_title)
        else:
            commit_message = '{} : v{}'.format(page_title, revision_counter)

        repo.commit(name=committer_name,
                    email=committer_email,
                    message=commit_message
                    )

        print('Committed {}'.format(commit_message))

        revision_counter += 1

    page_counter += 1

print('Processed', page_counter, 'pages')
```

## Cleaning up Output

```python
# -*- coding: utf-8 -*-

import sys
from glob import glob
import re

reload(sys)
sys.setdefaultencoding('utf-8')

for file in glob('./pages/*.md'):
    f = open(file, 'r').read()
    f_ = f

    f_ = f_.replace('\n', '')
    f_ = f_.replace('', '')
    f_ = f_.replace('<references />\n', '')
    f_ = f_.replace('<references />', '')
    f_ = f_.replace('`\\', '`')
    # f_ = re.sub(r'^`(( )+)?(.*)`', r'    \3', f_, flags=re.MULTILINE)
    f_ = f_.replace('` `', '')
    f_ = f_.replace('', '')
    f_ = f_.replace('`**', '')
    f_ = f_.replace('**`', '')
    f_ = re.sub(r'\s{4}(`\*\*`)', r'    ', f_, flags=re.MULTILINE)
    f_ = re.sub(r'(-\s{3})', r'* ', f_, flags=re.MULTILINE)

    with open(file, 'w') as o:
        o.write(f_)
        print 'Wrote', file

```

Then had to look at each manually :/

## Other Notes

### Comparisons

Flask using Markdown to render each page

```
Transactions:                349 hits
Availability:             100.00 %
Elapsed time:               9.20 secs
Data transferred:           3.21 MB
Response time:              1.77 secs
Transaction rate:          37.93 trans/sec
Throughput:             0.35 MB/sec
Concurrency:               67.14
Successful transactions:         349
Failed transactions:               0
Longest transaction:            2.27
Shortest transaction:           0.06
```

Statically generated HTML (as templates in Flask)

```
Transactions:               1671 hits
Availability:             100.00 %
Elapsed time:               9.02 secs
Data transferred:          15.37 MB
Response time:              0.01 secs
Transaction rate:         185.25 trans/sec
Throughput:             1.70 MB/sec
Concurrency:                1.23
Successful transactions:        1671
Failed transactions:               0
Longest transaction:            0.04
Shortest transaction:           0.00
```

Built-in Gollum server

```
Transactions:                 71 hits
Availability:              86.59 %
Elapsed time:               9.32 secs
Data transferred:           0.82 MB
Response time:              2.78 secs
Transaction rate:           7.62 trans/sec
Throughput:             0.09 MB/sec
Concurrency:               21.18
Successful transactions:          71
Failed transactions:              11
Longest transaction:            7.54
Shortest transaction:           0.00
```
