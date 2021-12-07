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

ABE student outcomes and required skills from job descriptions both characterize how a person is able to think and work. This project serves as an evaluation of these competencies. The main research question is: do ABE course competencies support students to meet job requirements? Answering this question will help curriculum directors determine what changes or supplementations should be made to the program. This keeps the department up to date with what employers are looking for.Evaluating the curriculum ensures that students are prepared to be successful and get their money's worth with a degree from ISU!

## Data Analysis

This section will provide a general outline of the approach taken to perform data analysis. The exact code utilized is located in this ![Jupyter Notebook](https://github.com/kjar0/516X-Final/blob/229f8b3c1e824566012866598d0a73502316967e/ABE_comp_job.ipynb). A project workflow is included at the conclusion of this section. The data analysis question is: how similar are the keywords of job postings to the keywords of course competencies?

#### Collect Information

To answer this research question, I received ABE competencies as well as graduating student survey answers from 2016-2021 from the department. From the survey answers, I retrieved the common job titles that ABE students have as well as the most popular employers. The top 5 job titles upon graduation that at least 3 people reported were:

> - Design Engineer
> - Project Engineer
> - Production Engineer
> - Agricultural Engineer
> - Manufacturing Engineer

These 5 job titles are good search terms to utilize because they represent all majors -- each option could work under 2+ of these titles. I used the common job titles as the search terms that would be input to indeed.com. 

For Indeed text scraping, I used a great code developed by Ryan Jeon as a starting point (thank you so much!). The base code performed text scraping of an indeed search for agriculture engineer that stored the job title, company, quick blurb posted on the home page, and url of the job posting. I needed to modify this code for the use I had in mind. I needed to go to the job posting for each job listed and extract the text of the entire job description. This is because a full list of the skills employers are looking for isn't typically shown in the blurb. I ultimately modified the source code into a function that would take a job title as the input and return a DataFrame containing job title, company, and URL for job posting. Then, for each saved URL from the first search, you navigate to that page and grab the job description text. This is added as another column on the DataFrame that is returned to the user. 

Even with a base code, accomplishing the text scraping from indeed was difficult due to the sheer volume. Once I started incorporating more jobs and search terms, I would get flagged as a bot. The links put together would require a CAPTCHA solve before going to the content. To address this, I utilized a number of User Agents where one was randomly selected for use with each query. I also added time delays to reduce the frequency of clicks/actions on the page as the code execution would be much faster than standard human use of Indeed.com. 

#### Pre-process data

Once I gathered the data, I needed to take the raw information and format it in a way that would be functional. The only changes I have made at this point are remove the \n (newline) characters from the job description text. I wrote a function that would take a string input and remove capitalization, punctuation, and common words that add little content value (the, a, if). After running the code a few times, I also created an addendum to the english common words that are removed from analysis. When running the analysis, the function will select key words that do appear frequently in the job postings, but don't provide a lot of context about job skills. Most of these job posting words I filtered out are related to logistic details about the job, for example insurance, pay, location, etc.

#### Data Manipulation & Analysis

Once I established the text scraping and formatting functions, I leveraged a 3rd function,`major_comp`, that would call the text scraping and formatting functions, then manipulate and analyze the data so the output could be easily visualized. To evaluate the text from the job posts of each job, I used the TF-IDF Vectorizer. This model takes string inputs and identifies the keywords from the document set based on the term frequency and inverse document frequency. Essentially it gives each keyword a score based on how relavent it is and how rare a word is in the entire document set. From these TF-IDF scores, I found the top 10 keywords for each job. Once I found all of these keywords, I created a dataframe to hold them that I output from the function. The second output from`major_comp` was a list of the keywords so they could be used for further analysis. 

Next, I used the TF-IDF Vectorizer again to assess the course outcomes against job post keywords. I processed the course outcomes using the same formatting function used for the job posts. Then, I trained the model using job keywords and fit the combined job keyword and filtered outcomes. I trained the model using job keywords because I wanted that to be the "vocabulary" or set of terms used to assess the course outcomes. 
```
# initialize vector and find TF-IDF matrix for each document
tfidf_v = TfidfVectorizer()
# train with vocabulary used in job postings
jfit = tfidf_v.fit(job_skill_4vec)
# transform job posting + competency key skills to TF-IDF matrix
tf_matrix = tfidf_v.transform(jobncomp_4vec)
```
I used the cosine_similarity function to compute the dot product of the job posting TF-IDF values and competency TF-IDF values. This is the measure of the similarity between two non-zero vectors in space.
```
# find cosine similarity for competency vs. posting by job type
csim = cosine_similarity(tf_matrix[0:1], tf_matrix)
```
![Heat Map](/images/csim_heat.jpg)

Finally, I added the top keywords for the common jobs and course competencies to a DataFrame for easy comparison. 

![Project Workflow](/images/flow.png)

## Discussion

A high cosine_similarity result indicates a closer match between two references. Based on this analysis of the 5 most common jobs for ABE graduates, the typical ABE student is most prepared for a design engineer role and least prepared for manufacturing engineering. The cosine_similarity output is shown in the heat map output above. Because the value corresponding to manufacturing engineering is the lowest, this indicates that the vectors are the most dissimilar of those analyzed in this dataset. As a whole, the job posting TF-IDF values were not as comparable to the course competencies as I expected. 

![Skill](/images/top_skills.jpg)

However, when looking at the top 10 keywords for each job description match very closely to course competencies for the ABE department. Because of this and the plethora of irrelevant information included in the job descriptions, I would conclude that by meeting the student outcomes, ABE graduates are adequately prepared with skills for the workforce.   

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
1. Update the text scraping function `indeed_posts` to take an additional input of keywords to narrow the search. This is currently set as the kword variable within the function, but make the necessary changes to allow the user to decide how to filter the search besides job title. Snippet of function relating to search url building shown below.

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

2. Identify another text modeling approach/processing step that can better account for the similarity of the words used in used phrases (ex. project & projects, systems & system, etc!)

![Python Notebook](ABE_comp_job.ipynb)
# make an assignment notebook here!!

## Conclusions and Final Thoughts

The ABE career outcomes data was very incomplete due to students not adding their employment information (job title, company). Further, not every student is completing the survey. As a student, I understand that at times it can be difficult to keep up with emails and fill out every survey when there's a lot on your plate. One way survey responses could be boosted is if this survey was added as an assignment for a senior-level class like Capstone II. This way, most people would be more motivated to complete the survey and the results would be more representative and therefore more useful.

Surprisingly, even when using the advanced search it was difficult to get agricultural/biological systems focused industry jobs. This could be a positive because not every graduate will stay in industry, but this could account for some of the discrepancy between the job posting text and competencies. Further, the job postings contain a decent amount of information relating to the company and various disclaimer statements about the job. Another area of improvement would be to have an even more exhaustive text preprocessing to remove that sort of information. Also, it would be helpful to utilize a method that identifies similar words (plurals, synonyms) while maintaining nuance. There are truly so many specialized options out there that there is significant room for improvement on this project. 
