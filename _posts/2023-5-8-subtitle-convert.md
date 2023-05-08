---
layout: post
title: "A subtitle conversion demo"
categories: study
author: "Jixiang Zhang"
---

[RAW format](https://www.kaltura.com/api_v3/service/attachment_attachmentasset/action/serve/attachmentAssetId/1_6hucdk4z?ks=djJ8MTQ0OTM2MnwX75vWYAiigw0NJ1TcVdEUf47uRKFjqsfZjj74hhM4tf-UcNAIDCuzRAwCSDkD2Xhns3SIim2bJ5BoW-hG7aARWBD4W3FthAQ-hjBAOWw839GfeQdU1fbKusZbB1_qxC89ExGhlOLkQ9viqBRc5jGLvsIRSalQIvfUtHgzP8PJQZH03U34Uhvel9qgLiqRYDtH1Ndv4ZYB2Gy0cYlIc77iJHrZp9TqlHBOX0daUGFYmZr0SUPWCBrm9Y9CNw-gIE6Rq1o1dihGsq1jiNYcyECdr10ATsVh8VnJPI0XGaQscjlVigwIrP6iC3H9GRooctuBAe5NUHbAQRUUcZiaqBSjw6j3ZDGE5tEYTzJDWa79RL31yo2YSo7XMsPy_VS47k5Yr7SmKz8nk2jjy-u4KB-4GVE_CLIDlTWEGUgpxRj37Xx720uVjVmNfIlqVoOhKt7DiJAq6uwRc2JzF_at0nAq0r_PtVDblB7eW4UFlNmRZDhvnJJs0tnKRQ5nC9-3mg9vWsWjJRsmnfIdNdUcLIbC) from ["Taskable Agility: Making Useful Dynamic Behavior Easier to Create" lecture by Scott Kuindersma](https://mediacentral.princeton.edu/media/%22Taskable+AgilityA+Making+Useful+Dynamic+Behavior+Easier+to+Create%22+lecture+by+Scott+Kuindersma/1_z4r6sz39)

```text
{
    "i": 0,
    "w": "All",
    "s": 6810,
    "e": 6869,
    "t": "word",
    "a": 55
},
{
    "i": 1,
    "w": "right.",
    "s": 6870,
    "e": 6990,
    "t": "word",
    "a": 56
}
```

Target [SRT format](https://en.wikipedia.org/wiki/SubRip)

```srt
1
00:00:06,810 --> 00:00:09,840
All right. I guess it gets onto the

2
00:00:09,870 --> 00:00:11,549
vehicles, but thanks a lot for for coming.
```

**Implementation**

```python
# %%
import json

f = open('data/subtitle_convert.json')
data = json.load(f)

# %%
import datetime


def time_format(ms):
    time_HMS = datetime.datetime.utcfromtimestamp(ms/1000).strftime('%H:%M:%S')
    time = time_HMS + "," + str(ms % 1000).zfill(3)
    return time


def create_one_subtitle(sub_index, s_time, e_time, sub_line):
    sub = str(sub_index) + "\n"
    sub += time_format(s_time) + " --> " + time_format(e_time) + "\n"
    sub += sub_line + "\n"
    return sub

# %%
sub_index = 1
s_time = 0
e_time = 0
sub_line = str()

is_start_time = True
word_count = 0
for i in data:
    if is_start_time:
        s_time = i["s"]
        is_start_time = False
    sub_line += i["w"] + " "
    word_count += 1
    if word_count > 7 or (word_count > 3 and (i["w"][-1] == '.' or i["w"][-1] == ',' or i["w"][-1] == '?')):
        e_time = i["e"]

        print(create_one_subtitle(sub_index, s_time, e_time, sub_line))

        sub_line = str()
        sub_index += 1
        word_count = 0
        is_start_time = True
```
