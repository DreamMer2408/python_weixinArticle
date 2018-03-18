import requests
from urllib.parse import urlencode
from requests.exceptions import ConnectionError
from pyquery import PyQuery as pq
import pymongo
from config import *

client=pymongo.MongoClient(MONGO_URL,27017)
db=client[MONGO_DB]


base_url='http://weixin.sogou.com/weixin?'
heads={
'Cookie':'SUID=4AF374B6541C940A00000000597418DC; SUV=00AA271EB674F373597ACBA5E9A18051; usid=qEzkWgGXp0Y5KSky; IPLOC=CN4109; LSTMV=176%2C400; LCLKINT=32846; teleplay_play_records=teleplay_616585:1; ABTEST=0|1520004943|v1; SNUID=109D18DB6C690B03D6D633526D59D2D0; weixinIndexVisited=1; JSESSIONID=aaarGXO8186WHdRm1zwhw; sct=3; ppinf=5|1520009114|1521218714|dHJ1c3Q6MToxfGNsaWVudGlkOjQ6MjAxN3x1bmlxbmFtZTo4OkRyZWFtTWVyfGNydDoxMDoxNTIwMDA5MTE0fHJlZm5pY2s6ODpEcmVhbU1lcnx1c2VyaWQ6NDQ6bzl0Mmx1SW9fUVZWSWk3bXU3V2xWRFlUNEtIOEB3ZWl4aW4uc29odS5jb218; pprdig=Mmyelhdg8C_3ORzPgz9pQMr395YxZ_7BPA9_bXc87APGGjQIUc94JI9aTHFfeMbkVbUaAjuRl7GoM54F6B5OH9SnLWNFQgg8IX6_CaE52O-xw2o3tzjNTteSsM4YAGuvS8MuyamxiH6qeQeAMhZqDQOiv8BOfbuD7y0s1aIL-n8; sgid=10-33882655-AVqZf5r5OgEmgXg5UrSYqjs; ppmdig=1520040953000000ab8a7a0a10a0ff0a8b7713ae7d494060',
'Host':'weixin.sogou.com',
'Referer':'http://weixin.sogou.com/weixin?query=%E9%A3%8E%E6%99%AF&_sug_type_=&sut=61884&lkt=1%2C1520008874181%2C1520008874181&s_from=input&_sug_=y&type=2&sst0=1520008874284&page=47&ie=utf8&w=01019900&dr=1',
'Upgrade-Insecure-Requests':'1',
'User-Agent':'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/55.0.2883.87 UBrowser/6.2.3964.2 Safari/537.36'
}

proxy=None
def get_index(keyword,page):
    data={
        'query':keyword,
        'type':2,
        'page':page,
        'ie':'utf8'
    }
    queries=urlencode(data)
    url=base_url+queries
    html=get_html(url)
    return html

def get_html(url,count=1):
    global proxy
    print('Trying count',count)
    print('conneting',url)
    if count>=MAXCOUNT:
        print('网确实崩了')
        return None
    try:
        if proxy:
            proxies={
                'http':'http://'+proxy
            }
            response=requests.get(url,headers=heads,allow_redirects = False ,proxies=proxies)
        else:
            response=requests.get(url,headers=heads,allow_redirects = False)
        if response.status_code==200:
            return response.text
        if response.status_code==302:
            print('302')
            proxy=get_proxy()
            if proxy:
                print('Using proxy',proxy)
                return get_html(url,count)
            else:
                print('Get proxy failed')
                return None
    except ConnectionError or TimeoutError:
        proxy=get_proxy()
        count+=1
        return get_html(url,count)

def get_proxy():
    try:
        response=requests.get(PROXY_POOL_URL)
        if response.status_code==200:
            print(response.text)
            return response.text
        return None
    except ConnectionError:
        return None

def parse_index(html):
    doc=pq(html)
    items=doc('.news-box .news-list li .txt-box h3 a ').items()
    for item in items:
        yield item.attr('href')

def get_detail(url):
    try:
        response=requests.get(url)
        response.encoding='utf-8'
        if response.status_code==200:
            return response.text
        return None
    except ConnectionError:
        return None
def parse_detail(html):
    doc=pq(html)
    title=doc('#activity-name').text()
    content=doc('.rich_media_content').text()
    date=doc('#post-date').text()
    nickname=doc('#meta_content a').text()
    wechat=doc('#js_profile_qrcode > div > p:nth-child(3) > span').text()
    return {
        'title':title,
        'content':content,
        'date':date,
        'nickname':nickname,
        'wechat':wechat
    }
def save_to_mongo(data):
    if db['articles'].update({'title': data['title']},{'$set': data},True):
        print('Save to Mongo',data['title'])
    else:
        print('Save to Mongo Failed',data['title'])
def main():
    for i in range(1,101):
        html=get_index('科比',i)
        if html:
            article_urls=parse_index(html)
            for article_url in article_urls:
                article_html=get_detail(article_url)
                if article_html:
                    article_data=parse_detail(article_html)
                    save_to_mongo(article_data)

if __name__ == '__main__':
    main()
