---
title: 耗时一周，我统计出来了npm下载量前30的仓库，第一竟是它！
date: 2023-03-18 16:19:03
categories: 
- 有趣
---

作为一个前端开发人员，我们每天都在使用npm，但是你曾经是否和我一样好奇，下载量最大的包是哪个？每天下载多少次？他们的github star是多少？上周我偶然看到了一个库 [glob](https://www.npmjs.com/package/glob)， 每周竟然下载8000万次，与此同时，react只有1500万次，glob是最高的吗，第一又是谁呢？

# 统计结果展示

耗时一周，我统计出来了npm下载量前30的仓库，第一竟是它！supports-color！总下载量 26,108,633,482 次，但 github star 竟然只有 319 个。另外，我做了一个网站，统计了最近一周、最近一月、最近一年、总下载量等各个维度的图表，还没有做优化加载可能有点慢，网站地址 https://www.npmrank.net/

无图无真相，下面是网站截图
![image](https://user-images.githubusercontent.com/35559153/226155971-98090186-88e2-4e45-a483-c66c4409e693.png)

![image](https://user-images.githubusercontent.com/35559153/226156027-c2c17c56-f661-41d2-969c-699cf17441d7.png)


# 分析npm官网接口，获取某个包的下载量

打开浏览器控制台分析npm接口发现，同一个地址，比如 https://www.npmjs.com/package/lodash ， 从npm首页 Popular libraries 中推荐的库点进去，接口返回的是JSON格式的数据，而从地址栏输入链接进去，返回是服务端渲染好的html。多次控制变量法未能定位是哪个header的原因，我就先不管了（当然不是睡大觉）
1. 找到返回JSON的接口，copy -> copy as fetch
![image](https://user-images.githubusercontent.com/35559153/226098795-aa0de0d3-bbbd-4357-ba5c-acf319b022e5.png)
2. 粘贴到console
![image](https://user-images.githubusercontent.com/35559153/226098836-f9629ddc-85f1-44f9-8410-79b6b8a4a0e2.png)
3. 复制header到postman，同时看到有下载量数据
![image](https://user-images.githubusercontent.com/35559153/226099349-3b354dbe-c606-40f3-8338-59f4377235e0.png)
4. 打开postman右侧的代码块，找到python代码
![image](https://user-images.githubusercontent.com/35559153/226099040-113c03f4-eb54-415a-950c-859070afc99d.png)
5. 复制到test.py，去掉某些空的header
![image](https://user-images.githubusercontent.com/35559153/226099169-c1f835e6-8fc6-4635-a426-d2a9725b2e5e.png)

OK，这样获取某一个仓库的接口就完成了，通过这个接口我们可以拿到github地址，仓库版本，最近一年每周的下载量等

# 根据npm官方api，获取不同时间段的下载量

上面官网的接口只是最近一年各周的下载量，有没有其他时间段的呢，找了一圈后发现npm官网提供了这样的接口，[官方api文档](https://github.com/npm/registry/blob/master/docs/download-counts.md) ，
![image](https://user-images.githubusercontent.com/35559153/226100591-6d6fce11-077f-405c-8ca9-9b415f6eeb96.png)
通过上面提供的接口，我们可以获取上周、上月、任何一个时间段的下载量，但是需要注意的是，官方api每次最多返回18个月的数据，最早是2015-01-10号的数据，所以统计总下载量时要分段获取每年的下载量后再累加，如果你想统计自己的包被安装了多少次，也是可以滴，接下来就是获取很多包名，循环下载后统计了

# 获取19年的排行

我在网上搜了一下npm download rank，发现只有 [anvaka](https://gist.github.com/anvaka/8e8fa57c7ee1350e3491) 19年做的统计符合想要的结果，他下载了npm全部的包并做了各种维度的分析，这个md是他统计的 top 1000依赖的包，不过被依赖的越多下载量越大，误差应该不会很大
![image](https://user-images.githubusercontent.com/35559153/226099718-5328c7de-d60e-4b83-ba57-348106bc99e0.png)
1. 保存文件到本地 SOURCE_FILE
2. 获取包名和仓库地址并存到sqlite数据库
    ```python
    '''
    从md中拿到库名并存到数据库
    '''
    with open(SOURCE_FILE, 'r') as f:
      lines = f.readlines()
      for line in lines:
        name = re.findall(r'\[(.*?)\]', line)
        href = re.findall(r'\((.*?)\)', line)
        print('line\n', line)
        if name and href:
          get_pkgbase_query = '''SELECT * FROM pkgbase WHERE id = ?'''
          record_base = sql_obj.get(get_pkgbase_query, (name[0],), one=True)
          if record_base is None:
            insert_data_query = '''
                                INSERT INTO pkgbase
                                ('id', 'npm_url', 'github_url', 'homepage_url', 'version', 'license', 'github_star', 'size', created, updated)
                                VALUES(?,?,?,?,?,?,?,?,?,?)
                                '''
            sql_obj.update(insert_data_query, (name[0], NPM_BASE_URL + name[0], '', '', '', '', 0, '', 0, 0))
    ```
    ![image](https://user-images.githubusercontent.com/35559153/226101055-90ffb9e0-65df-4395-b2b0-17359b1ef1a1.png)

3. 循环请求存储基本数据
    ```python
    '''
    更新下载量
    '''
    async def main():
      all_data_query = '''SELECT * FROM pkgbase'''
      records = sql_obj.get(all_data_query)
      for index, record in enumerate(records):
        while True:
          print('id', record['id'], index)
          try:
            '''
            获取下载量并写入数据库
            '''
            href = NPM_BASE_URL + record['id']
            npm_response = requests.request("GET", href, headers=npm_headers)
            npm_data = npm_response.json()

            # pkgbase
            github_url = npm_data['packageVersion'].get('repository', '')
            homepage_url = npm_data['packageVersion'].get('homepage', '')
            version = npm_data['packument'].get('version', '')
            license = npm_data['packument'].get('license', '')
            # 有仓库两个license
            license = license if type(license) == str else '-'
            versions = npm_data['packument'].get('versions') if npm_data['packument'].get('versions') else []
            updated = datetime.datetime.fromtimestamp(versions[0]['date']['ts'] / 1000).strftime("%Y-%m-%d %H:%M:%S")
            created = datetime.datetime.fromtimestamp(versions[len(versions) - 1]['date']['ts'] / 1000).strftime("%Y-%m-%d %H:%M:%S")

            update_pkgbase_query =  '''
                                    UPDATE pkgbase
                                    SET github_url = ?, homepage_url = ?, version = ?, license = ?, updated = ?, created = ?
                                    WHERE id = ?
                                    '''
            sql_obj.update(update_pkgbase_query, (github_url, homepage_url, version, license, updated, created, record['id']))
      ```
4. 更新各个时间段的下载量
    ```python
    '''
    获取某一时期的下载量
    '''
    def get_point_downloads(date_range, package_name):
      href = f'{NPM_BASE_API_POINT_URL}{date_range}/{package_name}'
      response = requests.request("GET", href)
      data = response.json()
      return data['downloads']

    '''
    获取全部下载量，npm每次最多返回18个月的数据，所以分段下载后再累加
    '''
    def get_point_all_downloads(package_name):
      start_time = 2015
      end_time = datetime.datetime.now().year
      all_downloads = 0

      for year in range(start_time, end_time + 1):
        dltype = f'{year}'
        date_range = f'{year}-01-01:{year + 1}-01-01'
        print('date_range', date_range)

        downloads = get_point_downloads(date_range, package_name)
        all_downloads += downloads
        print('new downloads',downloads)
        add_data_query = '''
                        INSERT INTO pkgdownload
                        ('id', 'dltype', 'downloads', 'timepoint')
                        VALUES(?,?,?,?)
                        '''
        sql_obj.update(add_data_query, (package_name, dltype, downloads, datetime.datetime.now()))
    return all_downloads
    
    ...

    # pkgdownload
    base_dltype = ['last-day', 'last-week', 'last-month', 'last-year', 'all-time']
    for dltype in base_dltype:
      if dltype == 'all-time':
        downloads = get_point_all_downloads(record['id'])
      else:
        downloads = get_point_downloads(dltype, record['id'])
      print('dltype', dltype)
      print('downloads', downloads)
      replaced_dltype = re.sub(r'\-', '_', dltype)
      add_pkgdownload_query =  '''
                                INSERT INTO pkgdownload
                                ('id', 'dltype', 'downloads', 'timepoint')
                                VALUES(?,?,?,?)
                                '''
      sql_obj.update(add_pkgdownload_query, (record['id'], replaced_dltype, downloads, datetime.datetime.now()))
    ```

# 获取包的github数据

本来官网接口中返回的有ghapi字段，如 https://api.github.com/repos/lodash/lodash ，里面有stargazers_count字段就是star数，但是该接口每小时限速60次，所以无奈只能用爬虫了，代码如下
```python
def set_github_info(github_url, package_name):
  response = requests.get(github_url, headers=github_headers)
  soup = BeautifulSoup(response.content, "html.parser")
  star = soup.find("span", class_='text-bold').get_text() if soup.find("span", class_='text-bold') else 0
  update_pkgbase_query =  '''
                            UPDATE pkgbase
                            SET github_star = ?
                            WHERE id = ?
                            '''
  print('package_name star', package_name, star)
  sql_obj.update(update_pkgbase_query, (star, package_name))
```
第一次使用爬虫库 bs4 的 BeautifulSoup 模块，获取 github star 只有两行代码，也太方便了吧

就在刚才发现npm也有接口会返回github star数，如 https://api.github.com/repos/lodash/lodash/pulls?per_page=1 里的 stargazers_count ，等有时间我替换一下

# 开启服务

经过上面一通操作，我们现在有了pkgbase、pkgdownload 这样两张表，内容如下
![image](https://user-images.githubusercontent.com/35559153/226102072-71e4b038-c492-4775-a8f8-768a70d9865d.png)
![image](https://user-images.githubusercontent.com/35559153/226102110-a6c1477a-9023-44c7-93d5-e22615c26f3c.png)

接下来写两个接口，一个是返回下载量排名的的类型，过去一周，过去一年，总下载量等，供前端筛选，使用quart简单起个服务

```python
from quart import Quart, request
import re

from db import SQLDB

app = Quart(__name__)
sql_obj = SQLDB()

'''
获取排名类型
'''
@app.route('/api/ranking/types')
async def get_types():
  return {
    'code': 200,
    'data': get_rank_types(),
    'success': True
  }

def get_rank_types():
  get_types_query = 'SELECT DISTINCT dltype FROM pkgdownload'
  records = list(map(convert_type, sql_obj.get(get_types_query)))
  
  return records

def convert_type(record):
    dltype = re.sub(r'\_', '-', record['dltype'])
    return {
      'label': dltype,
      'value': dltype
    }

if __name__ == '__main__':
    app.run(host='127.0.0.1', port=8080)
```
根据排名类型，返回对应的排行数据
![image](https://user-images.githubusercontent.com/35559153/226156536-7d15446d-13cd-4a25-b166-ff74d4716357.png)

```python
'''
获取包的数据
'''
@app.route('/api/ranking/packages/<path:rank_type>')
async def get_packages(rank_type):
  top = request.args.get('top')
  if top is None:
    top = 30
  elif int(top) > 200:
    top = 200
  else:
    top = int(top)
  rank_types = get_rank_types()
  rank_type =  next((c['value'] for c in rank_types if c['value'] == rank_type), None)

  if rank_type:
    rank_type = re.sub(r'\-', '_', rank_type)
    get_data_query =  '''
                      SELECT
                        a.id,
                        npm_url npmUrl,
                        github_url githubUrl,
                        homepage_url homepageUrl,
                        dltype dltype,
                        downloads downloads,
                        github_star githubStar,
                        version,
                        license,
                        updated,
                        created 
                      FROM
                        ( SELECT id, dltype, downloads FROM pkgdownload WHERE dltype = ? ORDER BY downloads DESC LIMIT 0, ? ) a,
                        pkgbase b
                      WHERE
                        a.id = b.id 
                      '''
    records = sql_obj.get(get_data_query, (rank_type, top))

    for index, record in enumerate(records):
      records[index]['rank'] = index + 1
    
    return {
      'code': 200,
      'data': records,
      'success': True
    }
```
![image](https://user-images.githubusercontent.com/35559153/226156582-af006bf7-d0df-454e-bce9-8cafdc6052be.png)

# 彩蛋

如果你看了上面开启服务的的代码，你可能会发现获取排行数据的接口其实还有一个top参数，最大是200条，但是由于图表不方便展示这么多的数据，如果你想自己看一下前200都有哪些包，可以复制接口改一下，如 https://www.npmrank.net/api/ranking/packages/last-day?top=200 ，如果你想查看超过200的排行，可以打开database.db的pkgdownload表查看

# 结束

以上就是获取npm排行的整个流程了，如果感觉有意思的话欢迎点个赞或者star，后端仓库地址 [npmrank](https://github.com/XueMeijing/npmrank) ，在线体验网页链接 https://www.npmrank.net/