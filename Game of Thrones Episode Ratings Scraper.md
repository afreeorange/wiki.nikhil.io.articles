```python
import re

from bs4 import BeautifulSoup
import requests


def get_rating(season, episode):
    endpoint = f"https://www.rottentomatoes.com/tv/game-of-thrones/s{season}/e{episode}"
    r = requests.get(endpoint)

    soup = BeautifulSoup(r.text, features="html.parser")
    g = soup.findAll("div", {"class": "tomato-info"})[0]
    p = g.findNext("div", {"class": "progress-bar"})

    return re.search(r"width\:(\d+)%;", p.attrs["style"])[1]


def make_episode_list(episodes):
    return [f"{_:02d}" for _ in range(1, episodes + 1)]


SEASONS_AND_EPISODES = (
    ("01", make_episode_list(10)),
    ("02", make_episode_list(10)),
    ("03", make_episode_list(10)),
    ("04", make_episode_list(10)),
    ("05", make_episode_list(10)),
    ("06", make_episode_list(10)),
    ("07", make_episode_list(7)),
    ("08", make_episode_list(6)),
)

for (season, episodes) in SEASONS_AND_EPISODES:
    for episode in episodes:
        try:
            rating = get_rating(season, episode)
        except Exception:
            rating = 'XX'
        print(f"S{season},E{episode},{rating}%")
```

### Results

```
S01,E01,100%
S01,E02,100%
S01,E03,86%
S01,E04,100%
S01,E05,95%
S01,E06,100%
S01,E07,100%
S01,E08,100%
S01,E09,100%
S01,E10,100%
S02,E01,100%
S02,E02,89%
S02,E03,100%
S02,E04,97%
S02,E05,96%
S02,E06,100%
S02,E07,96%
S02,E08,100%
S02,E09,100%
S02,E10,92%
S03,E01,98%
S03,E02,89%
S03,E03,92%
S03,E04,100%
S03,E05,100%
S03,E06,83%
S03,E07,77%
S03,E08,97%
S03,E09,100%
S03,E10,100%
S04,E01,95%
S04,E02,100%
S04,E03,96%
S04,E04,96%
S04,E05,98%
S04,E06,96%
S04,E07,100%
S04,E08,96%
S04,E09,94%
S04,E10,98%
S05,E01,96%
S05,E02,96%
S05,E03,100%
S05,E04,96%
S05,E05,96%
S05,E06,54%
S05,E07,83%
S05,E08,100%
S05,E09,83%
S05,E10,92%
S06,E01,86%
S06,E02,87%
S06,E03,87%
S06,E04,100%
S06,E05,98%
S06,E06,88%
S06,E07,98%
S06,E08,85%
S06,E09,98%
S06,E10,99%
S07,E01,93%
S07,E02,96%
S07,E03,89%
S07,E04,97%
S07,E05,95%
S07,E06,84%
S07,E07,87%
S08,E01,92%
S08,E02,88%
S08,E03,75%
S08,E04,57%
S08,E05,46%
```
