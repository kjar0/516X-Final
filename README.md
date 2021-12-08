# ABE Competencies and Workforce Skills

## Research Background and Question

The ABE department is fantastic at Iowa State University! The undergraduate program is ranked #2 in the nation while the graduate program is #1. This recognition is well deserved and is due to the excellent faculty and commitment to students. As part of this commitment, the department works hard to ensure that students are fine-tuning the skills they need during their time at Iowa State so they can be successful in the workplace. These key skills are called competencies, and are:

> 1. an ability to identify, formulate, and solve complex engineering problems by applying principles of engineering, science, and mathematics
> 2. an ability to apply engineering design to produce solutions that meet specified needs with consideration of public health, safety, and welfare, as well as global, cultural, social, environmental, and economic factors
> 3. an ability to communicate effectively with a range of audiences
> 4. an ability to recognize ethical and professional responsibilities in engineering situations and make informed judgments, which must consider the impact of engineering solutions in global, economic, environmental, and societal contexts
> 5. an ability to function effectively on a team whose members together provide leadership, create a collaborative and inclusive environment, establish goals, plan tasks, and meet objectives
> 6. an ability to develop and conduct appropriate experimentation, analyze and interpret data, and use engineering judgment to draw conclusions
> 7. an ability to acquire and apply new knowledge as needed, using appropriate learning strategies.

When reading over this list, the language almost sounds like a job description! For example, here's a job description from John Deere (the company that hires the most ABE graduates) for a Design Engineer (the most common job title for ABE graduates):

> - Analyze assignments and determine engineering specifications for complex problems or projects of moderate scope
> - Produce and/or evaluate possible design solutions to improve cost, quality and performance based on specialized knowledge of engineering applications
> - Collaborate with other functional engineering groups, supply management and/or supplier personnel
> - Compile and furnish necessary information (engineering decisions and reports of pertinent design analyses data) to document the design solution required for building of prototypes and adoption of the design
> - Provide technical support to marketing, manufacturing, quality and supply management organizations

ABE student outcomes and required skills from job descriptions both characterize how a person is able to think and work. This project serves as an evaluation of these competencies. The main research question is: do ABE course competencies support students to meet job requirements? Answering this question will help curriculum directors determine what changes or supplementations should be made to the program. This keeps the department up to date with what employers are looking for. Evaluating the curriculum ensures that students are prepared to be successful and get their money's worth with a degree from ISU!

## Data Analysis

This section will provide a general outline of the approach taken to perform data analysis. The full code utilized for this project is located in this ![notebook](ABE_comp_job.ipynb), please take a look if you're interested! The data analysis question is: how similar are the keywords of job postings to the keywords of course competencies?

![Project Workflow](/images/wflow.png)

#### Collect Information

To answer this research question, I loaded ABE competencies (shown in Research Background) as well as graduating student survey answers from 2016-2021 from an excel file to a DataFrame. The first thing I did with the data was identify any missing values. I was shocked to see how many fields were left blank on the survey, over 50% for Job Title and even more for Organization name and City. 
```
abe_survey = pd.read_excel('ABE Career Outcomes Data 2016-2021.xlsx')
print(abe_survey.isna().sum())
Degree Date                0
Degree Level               0
Major 1 at Graduation      0
Organization Name        196
Job Title                190
City                     193
State                    172
dtype: int64

blank = abe_survey.isna().sum()
print(blank['Job Title']/len(abe_survey))
0.5026455026455027
```
From the survey answers available, I retrieved the most popular employers as well as common job titles that ABE students have upon graduation. 
```
bse = abe_survey[abe_survey['Major 1 at Graduation'] == 'Biological Systems Engineering']
ag = abe_survey[abe_survey['Major 1 at Graduation'] == 'Agricultural Engineering']
print('TOP 10 EMPLOYERS FOR BSE AND AE \n \nBSE Common Employers:\n',bse['Organization Name'].value_counts().nlargest(10))
print('AE Common Employers: \n', ag['Organization Name'].value_counts().nlargest(10))

TOP 10 EMPLOYERS FOR BSE AND AE 
 
BSE Common Employers:
Cargill, Incorporated            6
Ardent Mills                     2
ISG                              2
ADM                              2
PepsiCo                          2
John Deere                       2
Merck                            1
DuPont Industrial Biosciences    1
Trinity Consultants              1
Solenis                          1
Name: Organization Name, dtype: int64

AE Common Employers: 
John Deere                                     16
Vermeer Corporation                            12
Henning Companies LLC                           5
AGCO Corporate Group                            5
Kuhn North America, Inc.                        5
Ag Leader Technology                            4
Kent Corporation                                3
Sage Ag LLC                                     3
CNH Industrial                                  3
USDA Natural Resources Conservation Service     3
Name: Organization Name, dtype: int64

bse_job = bse['Job Title'].value_counts().nlargest(5)
ag_job = ag['Job Title'].value_counts().nlargest(5)
print("Most popular BSE jobs:\n", bse_job)
print("Most popular AG ENGR jobs:\n", ag_job)
Most popular BSE jobs:
Design Engineer                      3
Production Engineer                  3
Production Management Engineer       2
Management Associate                 2
Supply chain management associate    1
Name: Job Title, dtype: int64
Most popular AG ENGR jobs:
Design Engineer           25
Project Engineer           9
Manufacturing Engineer     7
Agricultural Engineer      4
Test Engineer              3
```
I thought it was interesting that the most popular job title for both majors is Design Engineer. I decided to utilize the top 4 job titles for each major for equal representation in the job description search. With the top 4 jobs, I ensured that the job title selected was something that at least 2 ABE graduates have been employed as.
```
bse_job = bse['Job Title'].value_counts().nlargest(4).to_frame()
ag_job = ag['Job Title'].value_counts().nlargest(4).to_frame()
bse_joblist = bse_job.index.tolist()
ag_joblist = ag_job.index.tolist()
abe_joblist = bse_joblist + ag_joblist
# set() removes any duplicates from the list!
abe_joblist = list(set(abe_joblist))
abe_joblist
['Production Management Engineer',
 'Manufacturing Engineer',
 'Agricultural Engineer',
 'Management Associate',
 'Project Engineer',
 'Production Engineer',
 'Design Engineer']
```
These 7 job titles are good search terms to utilize because they represent all aspects of the ABE curriculum -- each option could work under 2+ of these titles. I used the common job titles as the search terms that would be input to Indeed.com's advanced search. 

For Indeed text scraping, I started with a great code developed by Ryan Jeon (thank you so much!). The base code performed text scraping of an indeed search for agriculture engineer that stored the job title, company, quick blurb posted on the home page, and url of the job posting. I needed to modify this code for the use I had in mind. I'll be going through the function in sections below, so even though the code will be broken up, it's all a part of the same indeed_posts function. 

- First I split the string passed to the function into it's separate words, then imported packages for html/timing/random number generation. DataFrames are initialized to hold the information gathered from each job posting: job title, company name, and URL of the full posting. 

```
def indeed_posts(search_term):
    kw = search_term.split(" ")
## *** a huge thank you to Ryan Jeon for developing the base of this indeed text scraping code, it has been modified for this application
    from bs4 import BeautifulSoup
    import requests
    import time
    import random
    df = pd.DataFrame(columns = ["Job_Titles"])
    df2 = pd.DataFrame(columns = ["Company"])
    df3 = pd.DataFrame(columns = ["URL"])
```
- Next, I set up components of the search string (key phrases to the major like agriculture, environment, water, etc.) and list of user agents outside of the loop. Then, the following code executes over the first 3 results pages of the Indeed advanced search. The search string is formed from the job title passed into the function and the page curreently being searched. A user agent is randomly selected and passed in with addional headers arguments when requesting the url. This helps avoid being recognized as a bot because the series of requests isn't coming from the same browser. Next the code combs through the BeautifulSoup html to find and save identified information. Also, there is a set wait time so the code doesn't execute extremely quickly. I also added time delays to reduce the frequency of clicks/actions on the page as the code execution would be much faster than standard human use of Indeed.com. 

```
    sind = f'https://www.indeed.com/jobs?as_and='
    kword = '&as_phr&as_any=biological%2C%20agriculture%2C%20food%2C%20environment%2C%20biofuel%2C%20fermentation%2C%20water%2C%20machinery%2C%20animal'
    end = '&as_not&as_ttl&as_cmp&jt=all&st&salary&radius=25&l&fromage=any&limit=10&sort=date&psf=advsrch&from=advancedsearch'
    user_agents = ['Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.93 Safari/537.36 Edg/96.0.1054.43',
              'Mozilla/5.0 (Windows NT 10.0; WOW64; Trident/7.0; rv:11.0) like Gecko',
              'Mozilla/5.0 (Windows NT 10.0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.93 Safari/537.36',
              'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:94.0) Gecko/20100101 Firefox/94.0']
    for pagenum in range(0,30,10):
        for word in range(0,len(kw)-1):
            search_url = sind + kw[word] + '%20'
        search_url = search_url + kw[len(kw)-1] + kword + end+'%20&start=' + str(pagenum)
        user_agent = random.choice(user_agents)
        headers = {
                'dnt': '1',
                'upgrade-insecure-requests': '1',
                'User-Agent': user_agent,
                'accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9',
                'sec-fetch-site': 'none',
                'sec-fetch-mode': 'navigate',
                'sec-fetch-user': '?1',
                'sec-fetch-dest': 'document',
                'accept-language': 'en-GB,en-US;q=0.9,en;q=0.8',
                }              
        r = requests.get(search_url, headers = headers)
        time.sleep(10*random.random())
        soup = BeautifulSoup(r.text, 'html.parser')
        titles = soup.select("h2 span") 
        # select all span tags under the umbrella of h2 tags 
        companies = soup.find_all(class_ = "companyName")
        URLs = soup.find_all('a', attrs = {'class' : 'tapItem'})
        for title in titles:
            titles_list = title.text
            df.loc[len(df.index)] = [titles_list]
            df = df[df.Job_Titles != "new"]
        for company in companies:
            company_list = company.text
            df2.loc[len(df2.index)] = [company_list]
        for URL in URLs:
            base = 'http://www.indeed.com'
            link = URL.attrs['href']
            new_URL = base + link
            df3.loc[len(df3.index)] = [new_URL]    
        ## *************************** end of code edited from Ryan Jeon's amazing work! :)
```
- This section of the code to navigate to the URL stored for each job posting and extract the text of the entire job description. This is because a full list of the skills employers are looking for isn't typically shown in the blurb and I wanted to evaluate all of the information. 

```
 jdesc =  pd.DataFrame(columns = ["Job Description"])
    for post in df3['URL']:
        user_agent = random.choice(user_agents)
        headers = {
                'dnt': '1',
                'upgrade-insecure-requests': '1',
                'User-Agent': user_agent,
                'accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9',
                'sec-fetch-site': 'none',
                'sec-fetch-mode': 'navigate',
                'sec-fetch-user': '?1',
                'sec-fetch-dest': 'document',
                'accept-language': 'en-GB,en-US;q=0.9,en;q=0.8',
                }
        r = requests.get(post.format(0), headers = headers)
        soup = BeautifulSoup(r.text, 'html.parser')
        time.sleep(5*random.random())
        job_description = soup.find('div',{'id':'jobDescriptionText'})
        jd = job_description.get_text(separator=" ") if job_description else "N/A"
        jdesc.loc[len(jdesc.index)] = [jd.strip()]   
    indeed = pd.concat([df, df2, df3, jdesc], axis=1, join='inner')
    print("Done scraping for ", search_term)
    return indeed
```
    The information returned from this function is a combined dataframe of job title, company, job posting URL, and full job description.

#### Pre-process data

At this point, we have a very large volume of text data. I was interested to see what words were the most frequent in the job description data
```
Counter(" ".join(y['Job Description']).split()).most_common(5)
[('and', 3021), ('to', 1648), ('the', 1437), ('of', 1301), ('in', 894)]
```
Most of the words in the descriptions are common words that add little meaning and value to our analysis. Let's clean this up! This function takes a string input and utilizes stereotypical English stopwords/punctuation as well as other identified stopwords. After running the code a few times, I created this addendum to the english common words that are removed from analysis. When running the analysis, the function will select key words that do appear frequently in the job postings, but don't provide a lot of context about job skills. Most of these job posting words I filtered out are related to logistic details about the job, for example insurance, pay, location, etc.
```
# function to clean up text (remove common value-minimal words, punctuation, capitalization)
def clean_post(text):
    from nltk.corpus import stopwords
    stopwords = stopwords.words('english')
    engr_stopwords = ['work', 'experience', 'job', 'ability', 'insurance', 'pay', 'location', 'requirements',
                 'required', 'skill', 'skills', 'years', 'release', 'engineer', 'preferred', 'new', 'may', 'us', 'office',
                 '3m', 'status', 'must', 'cemex', 'rfa', 'ula', 'position', 'benefits', 'including', 'gdcc', 'lockheed',
                 'martin', 'andor', 'apple', 'company', 'target', 'federal', 'western', 'education', 'effectively',
                 'appropriate', 'meet', 'acquire', 'first', 'needs', 'needed', 'mode', 'rf', 'nand', 'related',
                 'cae', 'amazon', 'general', 'baker', 'zero', 'nos', 'employees', 'make']
    stopwords.extend(engr_stopwords)
    text = ''.join([word for word in text if word not in string.punctuation])
    text = text.lower()
    text = ' '.join([word for word in text.split() if word not in stopwords])
    return text
```
This function returns a string stripped of punctuation and words identified as low-value for this analysis. 

#### Data Manipulation & Analysis

![mfunc](/images/major_comp.png)

Once I established the text scraping and formatting functions independently, I leveraged a 3rd function, `major_comp`, that would call the text scraping and formatting functions, then manipulate and analyze the data so the output could be easily visualized. 
```
def major_comp(joblist):
    from sklearn.feature_extraction.text import TfidfVectorizer
    import itertools
    tfidf_v = TfidfVectorizer()
    tskill = []
    dfs = {}
    for job in joblist:
        # retrieve job posting text
        search_results = indeed_posts(job)
        posts = search_results['Job Description'].tolist()
        # clean job postings
        clean_posts = list(map(clean_post, posts))
        # utilize TF-IDF vectorization and retrieve identified keywords
        eval_job = tfidf_v.fit_transform(clean_posts)
        fname = tfidf_v.get_feature_names()
        
        # creating dictionary of top skills
        df = pd.DataFrame(eval_job.T.toarray(), index=fname)
        job_avg = df.mean(axis=1)
        job_avg = pd.DataFrame(job_avg.sort_values(ascending=False).nlargest(10))
        # store name of keyword rather than associated TF-IDF score
        job_avg.index.name = 'Skill'
        job_avg.reset_index(inplace=True)
        dfs[job] = pd.DataFrame(job_avg['Skill'])

    # combine dictionary into dataframe
    skillsdf = pd.concat(dfs, axis=1)
    # column names = job title
    skillsdf = skillsdf.droplevel(1, axis = 1)
    skills4vec = []
    # loop through jobs
    for job in joblist:
        # convert job key skills to list
        a = skillsdf[job].tolist()
        # change from 10 strings to one long string!
        b = " ".join(a)
        # add to list of total skills, ready to use in Tfidf Vectorization
        skills4vec.append(b)    
    return skillsdf, skills4vec
```
This function retrieves job posting information for each job in list, cleans the posts, and finds the 10 keywords for the job title using TF-IDF. Once this is complete for each job in the list, it combines the 10 keywords from each job into a list of strings!


>> **What is TF-IDF?**
>>
>> The TF-IDF Vectorizer takes string inputs and identifies the keywords from the document set based on the term frequency and inverse document frequency. Essentially it gives each keyword a score based on how relevant it is and how rare a word is in the entire document set. 


Next, I used the TF-IDF Vectorizer again to assess the course outcomes against job post keywords. I processed the course outcomes using the same formatting function used for the job posts. Then, I trained the model using job keywords and fit the combined job keyword and filtered outcomes. I trained the model using job keywords because I wanted that to be the "vocabulary" or set of terms used to assess the course outcomes. This way, the TF-IDF scores represent how each document performs compared against the standard of job postings. In this analysis, we are most interesting in seeing how the course competencies perform. 
```
# initialize vector and find TF-IDF matrix for each document
tfidf_v = TfidfVectorizer()
# train with vocabulary used in job postings
jfit = tfidf_v.fit(job_skill_4vec)
# transform job posting + competency key skills to TF-IDF matrix
tf_matrix = tfidf_v.transform(jobncomp_4vec)
```
I used the cosine_similarity function to compute the dot product of the job posting TF-IDF values and competency TF-IDF values. This gives the measure of the similarity between two non-zero vectors in space.
```
# find cosine similarity for competency vs. posting by job type
csim = cosine_similarity(tf_matrix[0:1], tf_matrix)
print(csim)
sns.heatmap(csim)
array([[1.        , 0.23902804, 0.24906492, 0.30278205, 0.02753456,
        0.28130334, 0.23832191, 0.39331685]])
```
![Heat Map](/images/fhmap.png)

I added the top keywords for the common jobs and course outcomes to a DataFrame for easy comparison. Notice that only 4 keywords for the course outcomes were added. That is because the outcomes only contained 4 of the 29 keywords identified from the job postings.
```
df = pd.DataFrame(tf_matrix.T.toarray(), index=fname)
df = df.sort_values(by=0, ascending=False)
display(df)
df.index.name = 'Outcome'
df.reset_index(inplace=True)
job_skill_df['Outcome'] = pd.DataFrame(df['Outcome'][:4])
job_skill_df
```
![Matrix](/images/matrix.jpg)
![Skill](/images/daskills.jpg)

I also wanted to look at what skills were the most frequent in this dataframe.
```
import numpy as np
# dataframe --> single array of all job keywords
x = job_skill_df.drop(['Outcome'], axis=1).to_numpy()
kwlist =x.flatten()
# count frequency
Counter(kwlist).most_common(10)
[('engineering', 6),
 ('team', 6),
 ('project', 5),
 ('equipment', 5),
 ('quality', 4),
 ('design', 4),
 ('management', 4),
 ('manufacturing', 3),
 ('technical', 3),
 ('process', 3)]
```
3 keywords for curriculum outcomes (engineering, design, team) are also located within the top 6 keywords for all job descriptions! The 4th curriculum outcome keyword, knowledge, did not make the top 10 for job descriptions. 

## Discussion

#### Data Challenges

The ABE career outcomes data set was pretty incomplete due to students not adding their employment information (job title, company). Further, not every student is completing the survey. As a student, I understand that at times it can be difficult to keep up with emails and fill out every survey when there's a lot on your plate. One way survey responses could be boosted is if this survey was added as an assignment for a senior-level class like Capstone II. This way, most people would be more motivated to complete the survey and the results would be more representative and therefore more useful.

Surprisingly, even when using the advanced search tool on Indeed it was difficult to get agricultural/biological systems focused industry jobs. This could be a positive because not every graduate will stay in industry, but this could account for some of the discrepancy between the job posting text and competencies. More work could be done to fine tune this search query. Further, the job postings contain a decent amount of information relating to the company and various disclaimer statements about the job. Another area of improvement would be to have an even more exhaustive text preprocessing to remove that sort of information. Also, it would be helpful to utilize a method that identifies similar words (plurals, synonyms) while maintaining nuance. There are truly so many specialized options out there that could be utilized effectively. 

#### Drawing Conclusions

A high cosine_similarity result indicates a closer match between two references. Based on this analysis of the 5 most common jobs for ABE graduates, the typical ABE student is most prepared for a design engineer role and least prepared for a management associate role. The cosine_similarity output is shown in the heat map output above. Because the value corresponding to management associateg is the lowest, this indicates that the vectors are the most dissimilar of those analyzed in this dataset. As a whole, the job posting TF-IDF values were not as comparable to the course competencies as I expected. I would be interest to investigate the cause of this with further analysis. I assume it is because the TF-IDF Vectorization does not account for the similarity of words (team and teamwork, system and systems, etc.). It would also be interesting to see how the text cleaning could be improved by just extracting the job skills rather than the entire post with company information. 

Because of the low values, I'll be drawing conclusions mostly from the keywords. The three keywords that the course competencies fit match very highly with the job descriptions. Based on my work, I have concluded that curriculum outcomes are not the best source data to represent what students learn in the ABE department. These are written very broadly and make generalizations that can relate to various work areas. Job description verbiage tends to be more specific and action oriented rather than analysis based. In future iterations of this work, additional information that is more specific to each focus area could be added or even individual course descriptions. This would give the ABE outcomes a broader text base to work with.Just by looking at the top keywords for each job, it is my opinion that the ABE curriculum adequately prepares students with those skills. Although the verbiage isn't directly included in the competencies, through experience I know that many courses cover these topics. 

### Relevant Class Topics

- Ability to read in and incorporate packages
- Text scraping
- Data wrangling
- Exploratory data analysis

To complete this project, I needed to complete a lot of independent investigation into text analysis. I definitely went down a few rabbit holes... there's so many cool models to use that are definitely way above my level of understanding, so I'm sure that this analysis could be improved significantly. However, I did have to install a few packages and utilize them in this code (ex. NLTK). Text scraping was also a big component of this project. I got to experience the fun of combing through messy html looking for one small snippet! Also, I really enjoyed troubleshooting the CAPTCHA issues and figuring out how to go around text scraping filters. To no surprise, a significant amount of time on this project was spent messing around with data so I could actually use it! I realized how much I rely on DataFrames because I'm more familiar with the formatting and different actions you can take, but for future practice I'm going to work on manipulating different data types, particularly arrays and dictionaries. 

### FAIR principles 

Findable: This code and relevant data is published on github

Accessible: This code and relevant data is published on github

Interoperable: This code is decently interoperable. I have the key steps broken up into functions so a particular function could be extracted and used in another workflow. While the processing of the raw data is tailored to the file, any information to be used simply needs to be put into a list. Further, the text scraping function can be utilized for any job search. 

Reusable: The keywords identified from this analysis could be referenced with course objectives specific to each major's option to determine which specific options need strength. The figures produced from the heat mat aren't necessarily reproducable with the current state of the code because Indeed.com is constantly updating with new jobs. This could be changed with further editing by having a fixed set of job postings utilized. However, choice this keeps the results up to date with the job market. Also, the analysis just requires a list of strings. This section of the code could be used to compare any pieces of text!

## Task Suggestions
- Update the text scraping function `indeed_posts` to take an additional input of keywords to narrow the search. This is currently set as the kword variable within the function, but make the necessary changes to allow the user to decide how to additionally filter the search besides job title. For example, this could be by keyword but also location and salary. Snippet of function relating to search url building shown below.

```
def indeed_posts(search_term):
   #####
   code removed, see file
   #####
    sind = f'https://www.indeed.com/jobs?as_and='
    kword = '&as_phr&as_any=biological%2C%20agriculture%2C%20food%2C%20environment%2C%20biofuel%2C%20fermentation%2C%20water%2C%20machinery%2C%20animal'
    end = '&as_not&as_ttl&as_cmp&jt=all&st&salary&radius=25&l&fromage=any&limit=10&sort=date&psf=advsrch&from=advancedsearch'
    for pagenum in range(0,10,10):
        for word in range(0,len(kw)-1):
            search_url = sind + kw[word] + '%20'
        search_url = search_url + kw[len(kw)-1] + kword + end+'%20&start=' + str(pagenum)
        r = requests.get(search_url, headers = headers)
   #####
   code removed, see file
   #####
```
Note: for full functionality in the context of this analysis, the major_comp function would need to be updated also. For the purpose of this task, just copy the indeed_posts function independently and get that working!

- Identify another text modeling approach/processing step that can better account for the similarity of the words used in used phrases (ex. project & projects, systems & system, etc!)

![Link to Task Notebook](task.ipynb)