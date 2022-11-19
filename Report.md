**Here's What I Found about Chinese Students Planning to Study Abroad after Analyzing 540,000+ Lines of WeChat Chat History**

**Abstract**

As the admission requirements of schools and students’ conditions change every year along with global political and economic development, students often have new expectations for their application results and study abroad services. In this project, I performed a quantitative analysis on WeChat chat records between me and 118 students and their families to find out what Chinese students care about the most in studying abroad and how practitioners in the education industry could provide effective service to them.

**Content**

Introduction

Step 1: Data Collection - Exporting WeChat Chat History to Computer

Step 2: Data Cleaning - Combining Txt Files, Removing Non-Chinese Characters and Stop Words, Performing Text Segmentation

Step 3: Data Analysis - Word Frequency, Sorting, Generating List

Step 4: Data Visualization - Presenting Word Frequency in Wordcloud


**Introduction**

I started this project in 2021, when I had been working in international education for 5 years. Previously, I had written standard guides on FAQs in the application process, including templates commonly used when filling out online applications and suggestions that we can give students on topics such as research proposals and personal history statements, which was a handy qualitative guide for which I received positive feedback from a dozen colleagues, and, more importantly, an increase in our sales revenue and net promoter score (NPS).
 
However, as the admission requirements of schools and students’ conditions change every year along with global political and economic development, students often have new expectations for their application results and our services. When I communicated with them, I was curious to find out how my team and I could further improve our service and provide a better experience for our clients, both before and after they go to study overseas. I wanted to find out what they cared about the most and how to answer their questions effectively, this time adopting a quantitative approach.
 
WeChat chat history became the most ideal data for analysis, considering it is the major way for students and their families to communicate with us in China, far more popular than phone calls and emails. In this article, I will introduce the steps of my research and present my analysis result, which would inspire practitioners in the education industry with the similar goal of providing services efficiently to their clients and other stakeholders in order to achieve sustainable growth.

**Step 1: Data Collection - Exporting WeChat Chat History to Computer**
 
This was the most challenging step of the project, since WeChat only allows users to check chat history in its own app or transfer chat history from one device to another by creating a backup within the app. Nevertheless, there are several ways to obtain the chat history file. 

At first, I planned to transfer chat history from my Android phone to my Windows computer and use 3rd-party software like Root Explorer to get the data file and export chat history that I would like to analyze from the data file, but there were many steps that were rather time-consuming and would create an overwhelming workload, such as rooting the phone and decoding "EnMicroMsg.db," the SQLite3 database, using a key comprised of the IMEI and UIN of the phone. After studying multiple methods, I used my iPad and MacBook to do the following:

  1.	Backed up WeChat records from my Android phone to my iPad.
  2.	Backed up my iPad data (including WeChat chat history) to my MacBook using iTunes.
  3.	Obtained the "wxbackup" file from the address
      "<Disk>:\Users\<username>\Apple\MobileSync\Backup" (Mac)
      "<Disk>:\Users\<username>\AppData\Roaming\Apple Computer\MobileSync\Backup\" (Windows)
  4.	Extracted and exported chat records with my clients from the "wxbackup" file using 3rd-party software. The software I used was WechatExport-iOS (https://github.com/stomakun/WechatExport-iOS), which allowed me to select chat records with my clients from all chat records and export them to txt files.

In this way, I created a corpus of chat records ready for analysis.
  
**Step 2: Data Cleaning - Combining Txt Files, Removing Non-Chinese Characters and Stop Words, Performing Text Segmentation**

After exporting WeChat chat history to my computer, I acquired a corpus consisting of 118 .txt files, each containing chat history between me and one student, or between me and a WeChat group with one student and their family. Since it would be more efficient to process one file instead of 118 files in Python, I first merged the 118 files with the following codes in Terminal.

    <Disk>:
  
    cd <Disk>:<folder path with 118 files>
  
    type *.txt > all

Adding a .txt after the file "all" gave me a nice "all.txt" file for the following analysis. In this .txt file, there were many lines of chat records. Here’s how a typical line of a WeChat chat record is presented in the form of codes:

{
  
      "m_nsFromUsr" : "<username>",
  
      "m_uiMesLocalID" : <id>,
  
      "m_nsToUsr" : "",
  
      "m_uiCreateTime" : <time>,
  
      "m_uiMessageType" : <type>,
  
      "m_nsRealChatUsr" : "",
  
      "m_uiMesSvrID" : <id>,
  
      "m_nsContent" : "<content in Chinese characters>"
  
    }

It can be seen from the codes that there is much information unrelated to the topic to be studied in this project, except for the Chinese words, phrases, and sentences in the last line after "m_nsContent." Since the students are all Chinese who study around the world, the content of communication is all in Chinese, so it is safe to leave out information that is not Chinese.

In the following codes, I used regular expressions to check whether the contents of "all.txt " match the pattern [\u4E00-\u9FA5], which is a condition for judging whether they are Chinese or not. Characters that are Chinese ones would be saved to "chn.txt."

    import re
    my_file_path = '<folder path>/all.txt'
    save_file_path = '<folder path>/chn.txt'
    my_file = open(my_file_path, 'r', encoding='utf-8')
    cop = re.compile("[^\u4e00-\u9fa5^]")
    for line in my_file.readlines():
        string = cop.sub("", line)
        save_file = open(save_file_path, 'a', encoding='utf-8')
        save_file.write(string)
        save_file.flush()
        save_file.close()

After leaving out non-Chinese words, the next step is to remove Chinese stopwords because they are insignificant to the analysis of this project. In the following codes, I used the Chinese stopwords list from this website (https://github.com/goto456/stopwords) to remove stopwords from my .txt file.

    import pandas
    words_df=pandas.DataFrame({'segment':segment})
    words_df.head()
    stopwords=pandas.read_csv("stopwords.txt",index_col=False,quoting=3,sep="\t",names=['stopword'],encoding="gb18030")
    words_df=words_df[~words_df.segment.isin(stopwords.stopword)]

Now that I had removed stopwords, text segmentation was ready to be performed, since key words and word frequency in chat history would be the focus of the analysis. I used "Jieba" (Chinese for "to stutter"), a Chinese word segmentation module, to finish this step.

    import jieba
    import codecs
    file=codecs.open(u"chn.txt",'r',encoding='utf-8')
    content=file.read()
    file.close()
    segment=[]
    segs=jieba.cut(content) 
    for seg in segs:
        if len(seg)>1 and seg!='\r\n':
            segment.append(seg)

And now the data is clean and ready for the next step: analysis of word frequency.

**Step 3: Data Analysis - Word Frequency, Sorting, Generating List**

In this step, I ran the following codes to see word frequency and sort the words.

    words_stat=words_df.groupby(by=['segment'])['segment'].agg({"wordsfrequency":numpy.size})
    words_stat=words_stat.reset_index().sort_values(by="wordsfrequency",ascending=False)
    words_stat

Since I would like a file with clearer results for future analysis and presentation to related stakeholders, such as company supervisors, I added an extra step of generating word frequency files.

    result2txt=str(words_stat)  
    with open('list.txt','a') as file_handle:  
        file_handle.write(result2txt) 
        file_handle.write('\n') 

The results are shown in “list.txt,” as the following list showed. 
  
     segment    wordsfrequency
     9526       老师  4011
     4488       学校  2717
     6443       提交  1410
     8416       申请  1402
     6428      推荐信  1148
     11199      邮件   941
     9400       网申   912
     8451       电脑   889
     5879      成绩单   840
     11207      邮箱   821
     10496      谢谢   785
     5495       微信   775
     6540       收到   716
     6932       明天   658
     10289      证明   619
     538        不用   602
     6427      推荐人   599
     6918       时间   570
     11641      项目   549
     ...       ...   ...
     3222     名正言顺     1
     3224       名点     1
     7401       格好     1
     7400       根绝     1
     0          一万     1
     [11867 rows x 2 columns]
 
Words that rank in the top five are "老师(teacher)", "学校(school)", "提交(submit)", "申请(application)", and "推荐信(recommendation letters,)" with frequencies of 4011, 2717, 1410, 1402 and 1148 respectively, showing that submitting applications and recommendation letters are topics that play a vital role in students’ study abroad applications and that students pay extra attention to. Other topics worth noticing include "网申(online application)" that occurred 912 times, "成绩单(transcript)" that occurred 840 times, "证明(certificate)" that occurred 619 times, and "推荐人(recommender)" that occurred 599 times. Compared with topics that are less frequently seen in chat history, these words with high frequency and related topics, such as how to submit an online application and how to order transcripts that fit the requirements of schools, should be given more focus by education service providers.

Up until now, I have got a list of high-frequency words, and there is still one step to do before I report my findings to my supervisor, which is data visualization. I will elaborate on the visualization, conclusion, and my reflection on this project in the next part.

**Step 4: Data Visualization - Presenting Word Frequency in Wordcloud**
  
A word cloud that can show word frequencies is suitable for this project. Upon first glance, audiences can see the words and their frequency ratio with other words. Using the following code, I generated a word cloud using words in "list.txt."

    import matplotlib.pyplot as plt
    from scipy.misc import imread
    from wordcloud import WordCloud,ImageColorGenerator
    %matplotlib
    bimg=imread('<path of the shape the wordcloud would be in>')
    wordcloud=WordCloud(background_color="white",mask=bimg,font_path='<path of the font>')
    #wordcloud=wordcloud.fit_words(words_stat.head(4000).itertuples(index=False))
    words = words_stat.set_index("segment").to_dict()
    wordcloud=wordcloud.fit_words(words["wordsfrequency"])
    bimgColors=ImageColorGenerator(bimg)
    plt.axis("off")
    plt.imshow(wordcloud.recolor(color_func=bimgColors))
    plt.show()
  
![image](https://user-images.githubusercontent.com/118432046/202845775-74f011c5-2d3e-4c60-8746-2f9dd742f05d.png)

**Conclusion: Feedback, Impact, Limitation, and Future Work**
  
The analysis results, which contained word frequency and suggested topics that we can put more focus and effort on when providing services to students, were recognized and acknowledged by the director of the department and served as an essential reference and inspiration for both internal training sessions and daily work in the department and later in the company. I'd like to thank Fiona, who was my director at the time, because this project would not have been possible without our shared belief in the value of data analytics in education services, her unwavering support of my efforts to improve business performance, and the many internal resources she gave me to help me reach my goal and vision.
 
Regarding the sampling, the absolute prevalence of WeChat in China provided the project with sufficient chat records. Meanwhile, I ensured the sampling of students was balanced to the greatest extent possible: Because I've been collecting data for the past five years, I know that the students came from almost every major, including business, the arts and liberal sciences, natural science, and engineering.
 
In the future, I hope to gain more insights by dividing students into groups with varying expectations and behaviors and examining differences in their chat histories. For example, what topics do students with low GPAs (GPA below 3.0/4.0) or high GPAs (GPA above 3.6/4.0) care about the most, and what topics do students who take their undergraduate studies in mainland China or overseas care about the most? Answering these questions necessitates more effort on the part of education service providers in developing appropriate databases, which is one limitation of this project as well as my motivation to pursue a master's degree in business informatics in the coming years. By carefully collecting data with acute business sense and effective communication skills and adopting a combinative approach of quantitative analysis and qualitative explanation, I am confident I can provide insightful support for my future company and help it achieve outstanding business performance.
