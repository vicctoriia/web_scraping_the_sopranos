# Web scraping for beginners: 'The Sopranos'

## Introduction 
After a lot of research, I decided it was time to do my first "web scraping". Almost never the data we would like to work with comes in structured datasets like the ones available at Kaggle. Hence, the reason for doing web scraping, which is nothing more than a technique to collect data from the internet, to eventually make them structured and finally reach your initial goal, whatever it might be.


I decided that I would work on collecting information about one of my favorite series (and one of the best of all time): **'The Sopranos'**. The idea, then, is web scraping two websites, IMDb and Wikipedia, to be able to answer the following questions:

1. What is the IMDb rating of all episodes in each season? Which 3 episodes of the series have the highest ratings?
2. Which season had the highest average ratings?
3. Which season of the series has won the most awards?
4. Considering only the Emmy (main TV awards), in which categories was the series most nominated and awarded?
5. The highest rated seasons on IMDb, were the most awarded seasons as well?

## Web Scraping 
I will only present the web scraping process from the IMDb site (so this text doesn't get too big lol).

1.  **Importing important libraries**

When it comes to web scrapping, there are two important libraries for us to work with in python: requests and BeautifulSoup. The first is responsible for accessing the HTML of the page of interest and returning it as text. The second, in turn, is responsible for helping to extract exactly the data we are interested in from that page.
```sh
from bs4 import BeautifulSoup
import requests
from requests import get
```
2. **Extracting the data**

So I started by using the request library to extract the HTML from the page I chose (IMDb's page).
```sh
url = 'https://www.imdb.com/title/tt0141842/episodes?season=1'
response = get(url)
print(response.text[:250])
```
Now it's time to extract specific information from the page, using the BeautifulSoup library. Remember that 'response.text' is nothing more than the object we created above (already with the URL of the page of interest) and that 'html.parser' represents an indication that we want to do an analysis using python.
```sh
html_soup = BeautifulSoup(response.text, 'html.parser')
```
Now it is time to use our HTML skills, because we need to indicate exactly where the information we are interested in is. To do this, open the page and right-click on the **'inspect element'** option.

You will come across an HTML representation of all the information on your page. In my case, the next step was to look for where in this code the following information was: episode name, rating, number of voters, episode description, release date and episode number/season.

![Alt ou título da imagem](https://cdn-images-1.medium.com/max/1000/1*RbX5_swObmMbrDH64y_T-g.png)


**At this point you don't have a lot of options but to dig through the code until you find the information you are interested in**. Notice down there that I was able to find exactly the information about the episodes that I would like. Notice that the information is inside a 'div' (HTML tag that indicates an area of the page) and also that they have a so-called 'class' that names their elements.


![Alt ou título da imagem](https://cdn-images-1.medium.com/max/1000/1*S3y7NK3fKcHlzDevH_tdIA.png)

We will then use the **find_all** function from the **Beautiful_Soup** library to get this class specifically:
```sh
episode_containers = html_soup.find_all('div', class_='info')
```
Now (looking at the first episode only), let's see if I can, for example, find its IMDb rating, notice that it is presented like this:


![Alt ou título da imagem](https://cdn-images-1.medium.com/max/1000/1*IcwDmTp1lX5Cm6LjOEpANA.png)


So, to get it, you only need to specify its location:
```sh
episode_containers[0].find('span', class_='ipl-rating-star__rating').text.strip()
```
The same logic applies to all other information. But logically you don't want to waste time collecting the information one by one from each episode of each season, so it is interesting to build a way to do this automatically:
```sh
sopranos_episodes = []

response = get('https://www.imdb.com/title/tt0141842/episodes?                  season=' + str(sn))

page_html = BeautifulSoup(response.text, 'html.parser')
episode_containers = page_html.find_all('div', class_ = 'info')

for episodes in episode_containers:
  season = sn
  episode_number = episodes.meta['content']
  title = episodes.a['title']
  airdate = episodes.find('div', class_='airdate').text.strip()
  rating = episodes.find('span', class_='ipl-rating-  star__rating').text
  total_votes = episodes.find('span', class_='ipl-rating-star__total- votes').text
  desc = episodes.find('div',class_='item_description').text.strip()
  episode_data = [season, episode_number, title, airdate, rating,    total_votes, desc]
  
sopranos_episodes.append(episode_data)  
```
Finally, let's see the DataFrame:


![Alt ou título da imagem](https://cdn-images-1.medium.com/max/1000/1*rs8BxQhf17Wetz3ov8pMiA.png)



## Answering the suggested questions
After getting the data from IMDb, my next step was to extract the data from this Wikipedia page, to obtain all nominations and wins that the series had in several awards in which it competed. As I said before, I will not discuss here the process of Wikipedia web scraping nor will I dwell on the details of pre-processing the data, but if you are interested you can access my code. Anyway, after extracting the information from the awards, I came up with the following DataFrame:


![Alt ou título da imagem](https://cdn-images-1.medium.com/max/1000/1*cqI9UctRbQZd6c3ecnfyoQ.png)

Let's answer the initial questions:


1. What is the IMDb rating of all episodes in each season? Which 3 episodes of the series have the highest ratings?
![Alt ou título da imagem](https://cdn-images-1.medium.com/max/1000/1*OIKu34q8f1HImTX6KyxImg.png)


Interesting to note that The Sopranos has no rating below 8.0, **the standard really is high!**
The three episodes with the highest ratings on IMDb were: 
- S03E11 (9.7)= 'Pine Barrens'
- S05E12 (9.7) = 'Long Term Parking
- S06E20 (9.6)= 'The Blue Comet



2. Which season had the highest average ratings?

**The season that had the highest ratings, on average, was the fifth season**. But note that the difference is little with respect to the third and the last season. To be a fair comparison, it is important to note that the sixth season is the only one that has 21 episodes and not 13, and in my opinion the last season is the best of all!

![Alt ou título da imagem](https://cdn-images-1.medium.com/max/1000/1*yBq0O0uypqToNMOepUVmZg.png)


3. Which season of the series has won the most awards?


![Alt ou título da imagem](https://cdn-images-1.medium.com/max/1000/1*I7bjLN6EvbZJeWdf5I8d9Q.png)


As you can see, considering all the awards, the series had a total of 193 nominations and an incredible **83 wins!**
We can notice that the years of greatest nominations and wins were 2000 and 2001. The 2000 award, based on the premiere date of the episodes, refers to the second season of the series. **Therefore, the second season of the series won the most awards!**

4. Considering only the Emmy (main TV awards), in which categories was the series most nominated and awarded?

Now let's take a look just at the results of TV's most prestigious award ceremony, the 'Primetime Emmy Awards':


![Alt ou título da imagem](https://cdn-images-1.medium.com/max/1000/1*yImHVkTp93_bbiHkZnbQDw.png)

The series' record nominations are in the categories of best directing (Outstading Directing for a Drama Series) and best screenplay (Outsting Writing for a Drama Series), the latter being the most nominated category and also the winner. Without considering the technical categories (Primetime Creative Arts Emmy Award), 'The Sopranos' won 18 Emmys!.

As for the cast, James Gandolfini won every time he was nominated as best actor (3 times). The actress Edie Falco also won 3 awards, remembering that unlike the lead actor category, the lead actress category had nominations for other actresses of the series, however, Edie Falco was the only one to win.


5. The highest rated seasons on IMDb, were the most awarded seasons as well?

As we have seen, the highest rated seasons on average in IMDb were the fifth, third and sixth seasons, respectively. However, regarding the awards overall, the second season was the winner (followed by the third season). Therefore, there is a disagreement between the public and award ratings. Now, let's consider, therefore, only the Emmy:


![Alt ou título da imagem](https://cdn-images-1.medium.com/max/1000/1*ow45VmcrtVe7EY2D1EyzgA.png)

**Considering only the Emmy awards, the two most awarded seasons (both with 4 awards) were the fourth and fifth season**. In fact, the fifth season was the highest rated season on IMDb, so there is convergence in the opinion of the public and the awards. Interestingly, the fourth season was, on average, the worst rated by the public on IMDb!

**That's it, I hope you have enjoyed it!**


![Alt ou título da imagem](https://media3.giphy.com/media/RMxqGPaXWey2I/giphy.gif?cid=ecf05e4787b6tg08xfoeylex2u9ddj16n27e6jk6h315dnrf&rid=giphy.gif&ct=g)

## References 
https://towardsdatascience.com/scraping-tv-show-epsiode-imdb-ratings-using-python-beautifulsoup-7a9e09c4fbe5

https://towardsdatascience.com/are-series-getting-worse-over-time-b81886c376d1

https://www.dataquest.io/blog/web-scraping-python-using-beautiful-soup/

https://medium.com/analytics-vidhya/web-scraping-a-wikipedia-table-into-a-dataframe-c52617e1f451

