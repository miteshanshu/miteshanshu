#!/usr/bin/env python3
"""Refresh public GitHub profile assets. Python standard library only."""
import collections,datetime as dt,html,json,os,pathlib,re,urllib.request
from html.parser import HTMLParser
USER='miteshanshu'
ROOT=pathlib.Path(__file__).resolve().parents[1]
def fetch(url,api=True):
 headers={'User-Agent':USER+'-profile','Accept':'application/vnd.github+json' if api else 'text/html'}
 token=os.environ.get('GITHUB_TOKEN')
 if token and api:headers['Authorization']='Bearer '+token
 with urllib.request.urlopen(urllib.request.Request(url,headers=headers),timeout=30) as r:body=r.read().decode()
 return json.loads(body) if api else body
class Calendar(HTMLParser):
 def __init__(self):super().__init__();self.cells={};self.tips={};self.tip=None
 def handle_starttag(self,tag,attrs):
  a=dict(attrs)
  if 'data-date' in a:self.cells[a['data-date']]=a.get('id')
  if tag=='tool-tip':self.tip=a.get('for');self.tips[self.tip]=''
 def handle_data(self,data):
  if self.tip:self.tips[self.tip]+=data
 def handle_endtag(self,tag):
  if tag=='tool-tip':self.tip=None
 def days(self):
  result={}
  for day,key in self.cells.items():
   text=self.tips.get(key,'').strip();m=re.match(r'(\d+) contribution',text)
   if m:result[day]=int(m[1])
   elif text.startswith('No contributions'):result[day]=0
   else:raise RuntimeError('Unrecognized contribution cell '+day)
  return result

def collect():
 today=dt.datetime.now(dt.timezone.utc).date()
 url=f'https://github.com/users/{USER}/contributions?from={today.year}-01-01&to={today.isoformat()}'
 c=Calendar();c.feed(fetch(url,False));days=c.days()
 if today.isoformat() not in days:raise RuntimeError('Contribution calendar is incomplete')
 # Last 30 days can span two years.
 if (today-dt.timedelta(days=29)).year<today.year:
  prev=Calendar();prev.feed(fetch(f'https://github.com/users/{USER}/contributions?from={today.year-1}-01-01&to={today.year-1}-12-31',False));days.update(prev.days())
 latest=[{'date':(today-dt.timedelta(days=i)).isoformat(),'count':days[(today-dt.timedelta(days=i)).isoformat()]} for i in range(29,-1,-1)]
 anchor=today if days[today.isoformat()] else today-dt.timedelta(days=1);streak=0
 while days.get(anchor.isoformat(),0):streak+=1;anchor-=dt.timedelta(days=1)
 # Older calendars are only needed if the current streak crosses the available boundary.
 while anchor.year<today.year and anchor.isoformat() not in days:
  prev=Calendar();prev.feed(fetch(f'https://github.com/users/{USER}/contributions?from={anchor.year}-01-01&to={anchor.year}-12-31',False));days.update(prev.days())
  while days.get(anchor.isoformat(),0):streak+=1;anchor-=dt.timedelta(days=1)
 items=[];page=1
 while True:
  result=fetch(f'https://api.github.com/search/issues?q=author%3A{USER}+is%3Apr+is%3Amerged+is%3Apublic&per_page=100&page={page}')
  if result.get('incomplete_results'):raise RuntimeError('GitHub search returned incomplete results')
  items+=result['items']
  if len(items)>=result['total_count']:break
  if len(items)>=1000:raise RuntimeError('Merge search exceeds search limit')
  page+=1
 external=[x for x in items if x['repository_url'].split('/repos/')[1].split('/')[0].lower()!=USER.lower()]
 repos=[];page=1
 while True:
  batch=fetch(f'https://api.github.com/users/{USER}/repos?per_page=100&type=owner&page={page}');repos+=batch
  if len(batch)<100:break
  page+=1
 languages=collections.Counter()
 for repo in repos:
  if not repo['fork'] and not repo['private']:languages.update(fetch(repo['languages_url']))
 total=sum(languages.values())
 if not total:raise RuntimeError('No language bytes returned')
 percentages={k:round(v/total*100,1) for k,v in languages.most_common()}
 return {'date':today.isoformat(),'year':today.year,'streak':streak,'contributions':sum(n for d,n in days.items() if d.startswith(str(today.year)) and d<=today.isoformat()),'active30':sum(d['count']>0 for d in latest),'last30':latest,'external':len(external),'external_repos':len(set(x['repository_url'] for x in external)),'project_names':list(dict.fromkeys(x['repository_url'].split('/')[-1] for x in external)),'languages':percentages,'sources':[url,f'https://api.github.com/users/{USER}/repos',f'https://api.github.com/search/issues?q=author%3A{USER}+is%3Apr+is%3Amerged+is%3Apublic']}

import datetime as dt,html

def render(data,dark=False):
 bg='#0d1117' if dark else '#ffffff';ink='#e6edf3' if dark else '#1f2328';muted='#9198a1' if dark else '#59636e';border='#30363d' if dark else '#d1d9e0';track='#21262d' if dark else '#eff2f5';accent='#3fb950' if dark else '#1a7f37'
 heat=['#21262d','#0e4429','#006d32','#26a641','#39d353'] if dark else ['#eff2f5','#9be9a8','#40c463','#30a14e','#216e39'];o=[f'<svg xmlns="http://www.w3.org/2000/svg" width="880" height="430" viewBox="0 0 880 430" role="img" aria-label="GitHub public activity for miteshanshu, updated {data["date"]}"><rect width="880" height="430" fill="{bg}"/>']
 def t(x,y,s,size=14,weight=400,color=None):o.append(f'<text x="{x}" y="{y}" font-family="Arial, Helvetica, sans-serif" font-size="{size}" font-weight="{weight}" fill="{color or ink}">{html.escape(str(s))}</text>')
 def r(x,y,w,h,fill,stroke='none',rx=4):o.append(f'<rect x="{x}" y="{y}" width="{w}" height="{h}" rx="{rx}" fill="{fill}" stroke="{stroke}"/>')
 def panel(x,y,w,h,title):r(x,y,w,h,bg,border,6);t(x+18,y+28,title,14,600)
 t(1,26,'GitHub activity',20,600);t(647,25,'Updated '+data['date'],12,400,muted);panel(1,45,550,209,'Contribution activity · last 14 days')
 for i,d in enumerate(data['last30'][-14:]):
  x=19+i*34;y=111;n=d['count'];level=0 if not n else 1 if n<4 else 2 if n<7 else 3 if n<12 else 4;r(x,y,25,25,heat[level],rx=2);col=ink if not n else ('#e6edf3' if level<3 else '#0d1117') if dark else ('#1f2328' if level<3 else 'white');
 start=dt.date.fromisoformat(data['last30'][-14]['date']).strftime('%d %b');end=dt.date.fromisoformat(data['date']).strftime('%d %b');t(19,203,f'{start} - {end}',12,400,muted);t(365,203,f'{sum(d["count"]>0 for d in data["last30"][-14:])} active days',12,600,accent);t(19,233,'Each square is one day on the GitHub calendar.',11,400,muted);t(19,166,'Less',11,400,muted)
 for i,c in enumerate(heat):r(51+i*19,155,13,13,c,rx=2)
 t(151,166,'More',11,400,muted)
 for y,n,label,sub in [(45,data['streak'],'Current streak','Contribution days'),(156,data['contributions'],str(data['year'])+' contributions','Public calendar total')]:
  r(565,y,314,98,bg,border,6)
  if label=='Current streak':
   t(584,y+28,'Current streak',14,600);t(584,y+65,str(n)+' days',23,600,ink);t(683,y+65,'Contribution days',11,400,muted)
  else:
   t(584,y+41,n,34,600,accent);t(682,y+33,label,14,600);t(682,y+57,sub,11,400,muted)
 panel(1,267,550,119,'Merged outside my own repos');t(19,330,data['external'],32,600,accent);t(66,329,f'PRs across {data["external_repos"]} projects',14,600);t(19,356,' · '.join(data['project_names'][:4]),12,400,muted);panel(565,267,314,119,'Languages in public code')
 shown=list(data['languages'].items())[:2];other=round(100-sum(v for k,v in shown),1)
 for y,(name,pct) in zip([318,347],shown):
  t(584,y,name,12);t(830,y,str(pct)+'%',12,600);o[-1]=o[-1].replace('<text ','<text text-anchor="end" ');r(681,y-8,70,5,track,rx=2);r(681,y-8,70*pct/100,5,muted,rx=2)
 t(584,372,f'Other {other}% · byte share, not proficiency',10,400,muted);t(1,413,'Public GitHub data · calendar days follow GitHub · languages exclude forks',11,400,muted);o.append('</svg>');return ''.join(o)

if __name__=='__main__':
 data=collect();assets=ROOT/'assets';assets.mkdir(exist_ok=True)
 for theme in ['light','dark']:(assets/f'github-activity-{theme}.svg').write_text(render(data,theme=='dark'))
 (assets/'github-activity-data.json').write_text(json.dumps(data,indent=2)+'\n')
 print('Generated light and dark assets for '+data['date'])
