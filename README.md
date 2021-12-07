# ABE Competencies and Workforce Skills

You can use the [editor on GitHub](https://github.com/kjar0/516X-Final/edit/main/README.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

### Markdown

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

For more details see [Basic writing and formatting syntax](https://docs.github.com/en/github/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax).
Overall, I will be looking for the following:

* Introduction to a research question (what is the background on this subject; why does the question matter; who will it help; how has been attempted to be answered)

## Research Question

The ABE department is fantastic at Iowa State University. The undergraduate program is ranked X in the nation while the graduate program is X. This recognition is well deserved and is due to the excellent faculty and commitment to students. As part of this commitment, the department works hard to ensure that students are fine-tuning the skills they need to be successful in the workplace. These key skills are called competencies, and are:

This project serves as an evaluation of ABE and AST/ITEC competencies. The main research question is: do ABE course competencies meet job requirements? Answering this question will help curriculum directors determine what changes or supplementations should be made to the program. This keeps the department on the front of what employers are looking for.

* Clear data analysis question 

## Data Analysis

To answer this question, I needed to perform data analysis. From the department, I received ABE competencies as well as graduating student survey answers from 2016-present. From the survey answers, I retrieved the common job titles that ABE students have as well as the most popular employers. 

I used the common job titles as the search terms that would be input to indeed.com. For indeed text scraping, I used a great code developed by Ryan Jeon as a starting point. The base code performed text scraping of an indeed search for agriculture engineer that stored the job title, company, quick blurb posted on the home page, and url of the job posting. I needed to modify this code for the use I had in mind. I needed to go to the job posting for each job listed and extract the text of the entire job description. This is because a full list of the skills employers are looking for isn't typically shown in the blurb. I ultimately modified the source code into a function that would take a job title as the input and return a dataframe containing job title, company, URL for job posting, and job description.



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
