# ratemds 
class ratemds:
    '''A web scrapping module written using python beautifulsoup and Requests libraries
      specifically to extract information of physcians practicing in Stamford, CT (Namme, Gender, Rating, Ranking and Reviews) 
      from https://www.ratemds.com
      
      The physcicians information are saved as a .csv file
     
      To get the output use the use the method call below:
            
      Result= ratemds()
      Result.output()    
      '''
   
    
    
    import requests
    from bs4 import BeautifulSoup
    import pandas as pd   
   
    
    def __init__(self):
        
        generator = (self.url[0] + '?page=' + str(num) for num in range(2,14))
        for i in generator: self.url.append(i) 
            
        
    def __uniqdoc(self):
        import pandas as pd 
        
        docname = []
        gender = []
        starrating = []
        ranking = []
        address = []
        reviews = []
        
         
        for page in self.url:     #each page url  
            import requests
            from bs4 import BeautifulSoup
            
            requesturl = requests.get(page)   #retrieving the target website
            urlsoup = BeautifulSoup(requesturl.text, 'html.parser') #using html parser to structure the retrieved html script
            search = urlsoup.find_all('a', class_="search-item-doctor-link") #retrieves all doctors profile link from the page
        #return self.search
            for i in range(len(search)-1):
                homeurl = 'https://www.ratemds.com'
                profilelocator = search[i+1]['href']  #extracts the pointer to doctors profile
                targetlink = homeurl + profilelocator  #joins the homeurl to profilelocator to build link to doctors profile
                
                psoup = self.docProfileLink(targetlink)  #physician soup
   
     
                pname = self.physicianName(psoup)
                docname.append(pname)
            
                pgender = self.physicianGender(psoup)
                gender.append(pgender)
            
                prating = self.starRating(psoup)
                starrating.append(prating)
            
                pranking = self.specialtyRanking(psoup)
                ranking.append(pranking)
            
                paddress = self.physicianAddress(psoup)
                address.append(paddress)
            
                preview = self.physicianReview(psoup)
                reviews.append(preview)
            
        self.doc = pd.DataFrame(columns=['physicianName','physicianGender','starRating','specialtyRanking', 'physicianAddress', 'physicianReview'])
        self.doc['physicianName'] = docname
        self.doc['physicianGender'] = gender
        self.doc['starRating'] = starrating
        self.doc['specialtyRanking'] = ranking
        self.doc['physicianAddress'] = address
        self.doc['physicianReview'] = reviews
        
        return self.doc.to_csv('webscrapping2.csv',)#, webscrapping2.save()
         
        #return docname, gender, starrating, ranking, address, reviews

        
    def docProfileLink(self,targetlink):
        import requests
        from bs4 import BeautifulSoup

        
        uniqdocurl = requests.get(targetlink)
        uniqdocurlsoup = BeautifulSoup(uniqdocurl.text, 'html.parser')  #using html parser to structure the retrieved  html script for unique physician
        return uniqdocurlsoup        
           
    def physicianName(self, uniqdocurlsoup):
        namesearch = uniqdocurlsoup.find_all('h1', itemprop='name')
        pname = namesearch[0].get_text()
        return pname  #physician name
        
    def physicianGender(self, uniqdocurlsoup):
        gendersearch = uniqdocurlsoup.find_all('div', class_="search-item-info")
        return gendersearch[3].text
               
    
    def starRating(self, uniqdocurlsoup):
        ratingsearch = uniqdocurlsoup.find_all("span", class_="star-rating")
        return ratingsearch[0]['title']
    
    def specialtyRanking(self, uniqdocurlsoup):
        ranksearch = uniqdocurlsoup.find_all("div", class_="search-item-info")
        return ranksearch[2].get_text()
        
    def physicianAddress(self, uniqdocurlsoup):
        address = []
        addresssearch = uniqdocurlsoup.find_all('div', class_="search-item-info")
        for i in addresssearch[4].find_all('meta'): 
            address.append(i['content'])
        return address
        
    def physicianReview(self, uniqdocurlsoup):
        reviewsearch = uniqdocurlsoup.find_all('p',itemprop="reviewBody")
        reviews = []
        for i in reviewsearch:
            reviews.append(i.get_text('\n'))
        return reviews
    
    def output(self):
        return self.__uniqdoc()






