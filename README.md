# ABE Competencies and Workforce Skills


### Kate Jaros' Final Project for ABE 516X

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

Overall, I will be looking for the following:

* Introduction to a research question (what is the background on this subject; why does the question matter; who will it help; how has been attempted to be answered)

## Research Background and Question

The ABE department is fantastic at Iowa State University! The undergraduate program is ranked '#2 in the nation while the graduate program is '#1. This recognition is well deserved and is due to the excellent faculty and commitment to students. As part of this commitment, the department works hard to ensure that students are fine-tuning the skills they need during their time at Iowa State so they can be successful in the workplace. These key skills are called competencies, and are:

> 1. an ability to identify, formulate, and solve complex engineering problems by applying principles of engineering, science, and mathematics
> 2. an ability to apply engineering design to produce solutions that meet specified needs with consideration of public health, safety, and welfare, as well as global, cultural, social, environmental, and economic factors
> 3. an ability to communicate effectively with a range of audiences
> 4. an ability to recognize ethical and professional responsibilities in engineering situations and make informed judgments, which must consider the impact of engineering solutions in global, economic, environmental, and societal contexts
> 5. an ability to function effectively on a team whose members together provide leadership, create a collaborative and inclusive environment, establish goals, plan tasks, and meet objectives
> 6. an ability to develop and conduct appropriate experimentation, analyze and interpret data, and use engineering judgment to draw conclusions
> 7. an ability to acquire and apply new knowledge as needed, using appropriate learning strategies.

When reading over this list, the language almost sounds like a job description! For example, here's a job description from John Deere (the company that hires the most ABE graduates) for a Design Engineer (the most common job title for ABE graduates)

### !Double check what the most common job title is and add that here teehee!

ABE student outcomes and required skills from job descriptions both characterize how a person is able to think and work. This project serves as an evaluation of these competencies. The main research question is: do ABE course competencies support students to meet job requirements? Answering this question will help curriculum directors determine what changes or supplementations should be made to the program. This keeps the department up to date with what employers are looking for.Evaluating the curriculum ensures that students are prepared to be successful and get their money's worth with a degree from ISU!

* Clear data analysis question 

## Data Analysis

To answer this research question, I received ABE competencies as well as graduating student survey answers from 2016-2021 from the department. From the survey answers, I retrieved the common job titles that ABE students have as well as the most popular employers. 

I used the common job titles as the search terms that would be input to indeed.com. For indeed text scraping, I used a great code developed by Ryan Jeon as a starting point (thank you so much!). The base code performed text scraping of an indeed search for agriculture engineer that stored the job title, company, quick blurb posted on the home page, and url of the job posting. I needed to modify this code for the use I had in mind. I needed to go to the job posting for each job listed and extract the text of the entire job description. This is because a full list of the skills employers are looking for isn't typically shown in the blurb. I ultimately modified the source code into a function that would take a job title as the input and return a dataframe containing job title, company, URL for job posting, and job description.

```

competencies = pd.read_excel('competencies.xlsx')
abe_comp = competencies[competencies['Major']=='Engineering'].drop(['Option(s)', 'Major'], axis=1)

```


* Clear identification of data inputs

* Clear identification of analysis methods, including a well documented workflow figure.  

Additionally, I would like to see a written discussion of the following:

* Incorporation of at least three topics relevant to this class  - what from the class did you use in this project and why might it be useful for research projects like this?  What are the advantages and disadvantages?  Were there any assumptions or transformations needed?  Some topics discussed:  data wrangling; exploratory data analysis / summarizing data; identification of patterns and relationships; making predictions and decisions; text scraping; automation; scaling; randomization and bootstrapping; statistical analysis; ability to read and incorporate packages; supervised and unsupervised learning; cloud computing; using standard inputs; version control; how to manipulate multiple files with standard workflows; reproducible workflows, etc.

Relevant Class Topics:
- Data Wrangling
- Text Scraping
- Ability to read in and incorporate packages

* How much does your analysis attain the FAIR principles? 


For example, what is the ability to automate and reproduce your analysis (if the file input were to change, could this analysis be reproduced and how easily?)  - how will someone else reproduce this analysis?  Is the data stored somewhere?  Can I reproduce the figures easily?

* Creation of one assignment based on your dataset for the class to complete - one can think of this of a task or homework assignment based on your project.

Other things I will look for:

* Inclusion of statistical tools

* Publication of workflow in a version controlled manner (your code should be on github)

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/kjar0/516X-Final/settings/pages). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://support.github.com/contact) and we’ll help you sort it out.
