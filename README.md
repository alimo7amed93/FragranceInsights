# Fragrance Insights: Analyzing Perfume Trends with Web-Scraped Data

** _With Focus on the UAE Market_**

## Introduction

In today’s data-driven world, businesses and enthusiasts alike can leverage
data analysis to uncover trends and insights. This blog post details my
journey of analyzing perfume trends in the UAE using web-scraped data. We will
cover the entire process from data scraping and preprocessing to conducting a
thorough analysis and finally visualizing our findings through a consolidated
dashboard.  
**Technologies used:**_Python, Selenium, BeautifulSoup, JSON, Pandas, Tableau,
Jupyter._

# Part 1: Data Scraping

**Objective** : Collect detailed information on perfumes available in the UAE
from various web sources.  
**_Tools Used_** : Python, BeautifulSoup, _Selenium, JSON_

**Process** :

  1. Identify Data Sources: I targeted one of the world’s most popular perfume websites: “Fragrantica” for comprehensive data on different perfumes from UAE.
  2. Extract Data: Using Python libraries like Selenium, BeautifulSoup and JSON, I extracted information such as perfume names, designers, ratings, gender, notes, and accords.
  3. Store Data: The scraped data was stored in a structured format (JSON/CSV) for further processing.

**Challenges:  
** Due to web scraping limitations, I had to use Selenium for WebDriver action
automation for scrolling, pressing buttons for more details. Nevertheless, I
was able to only collect data of 330 perfumes having in consideration the
nature and time for this personal study project.

> **> > Getting list of all perfumes targeted for scraping:**
    
    
    from bs4 import BeautifulSoup  
    from selenium import webdriver  
    from selenium.webdriver.chrome.service import Service  
    from selenium.webdriver.chrome.options import Options  
    from selenium.webdriver.common.by import By  
    from webdriver_manager.chrome import ChromeDriverManager  
    from pprint import pprint  
    import time  
    import json  
      
    #instantiate the browser driver  
    s=Service(ChromeDriverManager().install())  
    o=Options()  
    o.add_argument("start-maximized")  
    driver = webdriver.Chrome(service=s, options=o)  
      
    #Search URL with UAE as a filter  
    url = "https://www.fragrantica.com/search/?country=United%20Arab%20Emirates"  
    driver.get(url)  
    time.sleep(10)  
      
    # click load more results to load more perfume details  
    for i in range(0,10):  
        try:  
            driver.find_element(By.XPATH,'//button[normalize-space()="Show more results"]').click()  
            time.sleep(10)  
        except Exception as e:  
            print("An exception occurred:", e)  
            break  
      
    # Get the page html   
    html = driver.page_source  
    soup = BeautifulSoup(html, "html.parser")  
      
    # Get the list of perfume names  
    perfumeGrid = soup.find("span", class_="grid-x grid-margin-x grid-margin-y small-up-3 medium-up-2 large-up-4 perfumes-row text-center")  
    DictsList = []  
    Dict = {}  
    for str in perfumeGrid.find_all("a", href=True):  
        perfumeName = str.get_text().replace('\n', '').strip(" ")  
        perfumeURL= str['href']  
        Dict = {"name": perfumeName,  
                "url": perfumeURL  
                }  
        DictsList.append(Dict)  
      
    # Open a file in write mode and add the DictsList to it  
    with open('DictsList.txt', 'w') as file:  
        json.dump(DictsList, file)  
    time.sleep(3)  
    driver.quit()  
      
    # printing 1st perfume details to see structre  
    pprint(DictsList[0])  
      
    #printing count of loaded perfumes  
    DictsList = []  
    with open('DictsList.txt', 'r') as file:  
        DictsList= json.load(file)  
    print(len(DictsList))

> **> > Looping over each perfumes and scraping it’s data**
    
    
    #Get in each perfume's page to get the it's details  
    perfumesDictsList=[]  
    for i in range(len(DictsList)):#update this later len(DictsList)  
        try:  
            driver = webdriver.Chrome(service=s, options=o)  
            driver.get(DictsList[i]["url"])  
            time.sleep(10)  
        except:  
            break  
        html = driver.page_source  
        soup = BeautifulSoup(html, "html.parser")  
    ###################get the URL####################  
        perfumeUrl=DictsList[i]["url"]  
      
    ################get the Name######################  
        perfumeName = soup.find_all("div", class_="cell small-12")[3].find_all("b")[0].get_text()  
      
    ################get the designer##################  
        perfumeDesigner = soup.find_all("div", class_="cell small-12")[3].find_all("b")[1].get_text()  
      
    ######################get the img#################  
        perfumeImage = soup.find_all("div", class_="cell small-12")[1].find("img")["src"]  
      
    ######################get the gender##############  
        perfumeGender = soup.find("small").get_text()  
        try:  
            perfumeRating = float(soup.find("p", class_="info-note").find_all("span")[0].get_text())  
        except:  
            perfumeRating = "Null"  
      
    ######################get the Votes count#########  
        try:  
            perfumeVotesCount = int(soup.find("p", class_="info-note").find_all("span")[2].get_text().replace(',', ''))  
        except:  
            perfumeVotesCount = "Null"  
      
    ##################find the description############  
        try:  
            perumeDescription = soup.find_all("div", class_="cell small-12")[3].get_text().split('Read about this perfume')[0]  
        except:  
            perumeDescription = "Null"  
      
    ##################find the accords###############  
        try:  
            perfumeAccords = soup.find_all("div", class_="cell accord-box")  
            perfumeAccordsDict = {}  
            for i in range(len(perfumeAccords)):  
                accordName = perfumeAccords[i].get_text()  
                accordValue = float(perfumeAccords[i].find("div", class_="accord-bar")["style"].rsplit("width: ")[1].strip("%;"))  
                perfumeAccordsDict[accordName] = accordValue  
        except:  
                perfumeAccordsDict = {}  
      
    #################find the notes#################  
        perfumeNotes = soup.find_all("div", attrs={"style": "display: flex; justify-content: center; text-align: center; flex-flow: wrap; align-items: flex-end; padding: 0.5rem;"})  
        if len(perfumeNotes) == 3:  
            i = 2  
            perfumeTopNotes = []  
            perfumeMidNotes = []  
            perfumeBaseNotes = []  
            for j in range(len(perfumeNotes[0].find_all("span", class_="link-span"))):  
                perfumeTopNotes.append(perfumeNotes[0].find_all("div")[i].get_text())  
                i += 3  
            i = 2  
            for j in range(len(perfumeNotes[1].find_all("span", class_="link-span"))):  
                perfumeMidNotes.append(perfumeNotes[1].find_all("div")[i].get_text())  
                i += 3  
            i = 2  
            for j in range(len(perfumeNotes[2].find_all("span", class_="link-span"))):  
                perfumeBaseNotes.append(perfumeNotes[2].find_all("div")[i].get_text())  
                i += 3  
        elif len(perfumeNotes) == 2:  
            i = 2  
            perfumeTopNotes = []  
            perfumeMidNotes = []  
            perfumeBaseNotes = []  
            for j in range(len(perfumeNotes[0].find_all("span", class_="link-span"))):  
                perfumeTopNotes.append(perfumeNotes[0].find_all("div")[i].get_text())  
                i += 3  
            i = 2  
            for j in range(len(perfumeNotes[1].find_all("span", class_="link-span"))):  
                perfumeMidNotes.append(perfumeNotes[1].find_all("div")[i].get_text())  
                i += 3                
        elif len(perfumeNotes) == 1:  
            i = 2  
            perfumeTopNotes = []  
            perfumeMidNotes = []  
            perfumeBaseNotes = []  
            for j in range(len(perfumeNotes[0].find_all("span", class_="link-span"))):  
                perfumeMidNotes.append(perfumeNotes[0].find_all("div")[i].get_text())  
                i += 3        
        else:  
            perfumeTopNotes = []  
            perfumeMidNotes = []  
            perfumeBaseNotes = []  
      
    #################find the Max voting#################  
        voting = soup.find_all("div", class_="cell small-1 medium-1 large-1")  
        # Define dictionaries to map index to label for each category  
        labels = {  
        "Longevity": {0: "very weak", 1: "weak", 2: "moderate", 3: "long lasting", 4: "eternal"},  
        "Sillage": {0: "intimate", 1: "moderate", 2: "strong", 3: "enormous"},  
        "Gender": {0: "female", 1: "more female", 2: "unisex", 3: "more male", 4: "male"},  
        "PriceValue": {0: "way over", 1: "over", 2: "ok", 3: "good", 4: "great"}  
        }  
        maxLabels = {}  
        # Iterate over each category  
        for category, labelMap in labels.items():  
            # Find the index of the maximum value in the voting list for the current category  
            maxIndex = max(range(len(labelMap)), key=lambda i: int(voting[i].get_text()))  
            # Retrieve the corresponding label from the label map  
            maxLabel = labelMap[maxIndex]  
            # Store the category and its corresponding label with maximum votes in the maxLabels dictionary  
            maxLabels[category] = maxLabel  
      
    ############prepare the json and write it to the file###############  
    # creating each perfume's data in a json object  
        perfumeDict = {  
                    "url": perfumeUrl,  
                    "name": perfumeName,  
                    "desiger": perfumeDesigner,  
                    "image": perfumeImage,  
                    "gender": perfumeGender,  
                    "rating": perfumeRating,  
                    "votes count": perfumeVotesCount,  
                    "description": perumeDescription,  
                    "accords": perfumeAccordsDict,  
                    "top notes": perfumeTopNotes,  
                    "mid notes": perfumeMidNotes,  
                    "base notes": perfumeBaseNotes,  
                    "max votes": maxLabels  
                }  
        perfumesDictsList.append(perfumeDict)  
    # writing a copy of the perfume object to the json file  
        with open('perfumesDictsList.txt', 'w') as file:  
            json.dump(perfumesDictsList, file)  
        driver.quit()  
        time.sleep(10)  
      
    # printing 1st perfume details to see structure  
    pprint(perfumesDictsList[0])

# Part 2: Data Preprocessing

**Objective** : Clean and transform the scraped data to ensure consistency and
accuracy for analysis.

**Steps** :

  1. Standardize Field Names: Ensure all field names are consistent (e.g., correcting typos).
  2. Handle Missing Values: Fill in or remove missing data points.
  3. Convert Data Types: Ensure numeric fields like ratings and votes count are in the correct format.
  4. Flatten JSON Fields: Convert nested JSON fields into separate columns for easier analysis.

    
    
    import pandas as pd  
    # Load the data from file  
    with open('perfumesData.txt', 'r') as file:  
        data = json.load(file)  
      
    # Create a DataFrame  
    df = pd.DataFrame(data)  
      
    # Standardize column names  
    df.columns = [col.lower().replace(' ', '_') for col in df.columns]  
      
    # Rename any specific columns to maintain consistency  
    df.rename(columns={'votes_count': 'votes_count', 'desiger': 'designer'}, inplace=True)  
      
    # Handle missing values  
    df.fillna({'rating': 0, 'votes_count': 0}, inplace=True)  
      
    # Convert ratings and votes_count to appropriate types  
    df['rating'] = df['rating'].astype(float)  
    df['votes_count'] = df['votes_count'].astype(int)  
      
    # Extract top, mid, and base notes into individual columns  
    df['top_notes'] = df['top_notes'].apply(lambda x: ', '.join(x))  
    df['mid_notes'] = df['mid_notes'].apply(lambda x: ', '.join(x))  
    df['base_notes'] = df['base_notes'].apply(lambda x: ', '.join(x))  
      
    # Extract accords into individual columns  
    accords_df = pd.json_normalize(df['accords'])  
    accords_df.columns = [f'accord_{col}' for col in accords_df.columns]  
    # Convert accords values to float and round to 2 decimal points  
    accords_df = accords_df.astype(float).round(2)  
    # Merge accords back into the main DataFrame  
    df = pd.concat([df.drop(columns=['accords']), accords_df], axis=1)  
      
    # Flatten the nested dictionary into separate columns  
    max_votes_df = pd.json_normalize(df['max_votes'])  
    # Concatenate the flattened DataFrame with the original DataFrame  
    df = pd.concat([df.drop(columns=['max_votes']), max_votes_df], axis=1)  
      
    # Handle missing values  
    df.fillna(0, inplace=True)  
      
    # Drop unneaded columns  
    df.drop(columns=['url','image', 'description'],inplace=True)  
      
    # Save preprocessed data to a new file  
    df.to_csv('preprocessed_perfumesData.csv', index=False)  
      
    # Print summary to check the preprocessing  
    df.head()

# Part 3: Data Analysis

**Objective** : Derive insights from the cleaned data using various analysis
techniques.  
**_Tools Used_** : Tableau, Excel

**Questions Explored** :

  1. **_Distribution of Ratings:_**

  * Question: What is the distribution of ratings among different perfumes?
  * Visualization: Histogram of perfume ratings.

2\. **_Top-Rated Perfumes:_**

  * Question: Which perfumes have the highest ratings?
  * Visualization: Bar chart of perfumes by rating.

3\. **_Votes Count Analysis:_**

  * Question: How many votes did each perfume receive?
  * Visualization: Bar chart or scatter plot of votes count vs. perfume names.

4\. **_Gender Distribution:_**

  * Question: How are perfumes distributed across different genders?
  * Visualization: Pie chart or bar chart of gender distribution.

5\. **_Accords Analysis:_**

  * Question: What are the most common perfumes having certain accord?
  * Visualization: Bar chart of the frequency of different accords.

6\. **_Price-Value Perception:_**

  * Question: How do users -based on most common votes- perceive the price-value of different perfumes ?
  * Visualization: Highlight table by their price-value ratings.

7\. **_Longevity and Sillage:_**

  * Question: What are the longevity and sillage -based on most common votes- characteristics of perfumes?
  * Visualization: Grouped bar chart showing longevity and sillage votes.

# Part 4: Tableau Visualization Dashboard and Setup

**Data Import** :

  * Import the cleaned data CSV file into Tableau.

**Creating Visualizations** :

  * Drag and drop the relevant fields into the Columns and Rows shelves to create the desired charts.
  * Use Filters to narrow down the data if needed (e.g., filter by designer or gender).

**Dashboards** :

  * Combine multiple charts into a single dashboard for a comprehensive view.

**Interactivity** :

  * Add interactivity (filters, highlights) to allow users to explore the data dynamically.

# Conclusion

By following the above preprocessing steps and setting up the suggested
visualizations, we will be able to gain deep insights into your UAE perfumes
data. Tableau’s powerful visualization capabilities will help us present these
insights in an engaging and informative manner.

Some of the Findings:
- The top 3 rated perfumes are “Kayaan Classic”, “Al Nashama Caprice”, and “Sharaf Blend” respectively.
- The most common rating values are between 4.08 to 4.28 which is given to 42% of the perfumes
- Majority of votes count are less than 2k. Only 3 perfumes exceeded 4k votes, they are namely: “Vanilla 28”, “Hawas for him”, and “Asad”.
- Highest percentage of perfume gender percentage is Unisex at 58%.
- Majority of votes are going towards enormus sillage: 69% and long lasting rather than eternal at 66% of the total votes

**_GitHub link:  
_**<https://github.com/alimo7amed93/FragranceInsights>

