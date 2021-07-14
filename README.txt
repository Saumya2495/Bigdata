README

#################################################################
General Information

 - Program language: Python

 -Python Library: tika, spacy, nltk, json, pandas, os, re, dateutil, bs4, spellchecker, textblob, geonamescache, datetime, urllib

 - External resource: spacy nlp pipeline: en_core_web_lg, API from ipstack.com,  API from scamalytics.com, tika-similarity, blogtext.csv from Rachael Tatman, jaccard.py, edit.py, cosine.py and circle-packing-for-all.py from Jiarui Ou

 - External dataset files: 
   * earthquakes-2021-03-08_20-15-27_-0600.tsv (external dataset1)
   * tsunamis-2021-03-08_19-39-11_-0600.tsv (external dataset1) 
   * API_SP.POP.DPND_DS2_en_xml_v2_2059837.xml (external dataset2)
   * API_IT.NET.USER.ZS_DS2_en_xml_v2_2058003.xml (external dataset2)
   * API_NY.GDP.PCAP.KD_DS2_en_xml_v2_2057054.xml (external dataset2)
   * URL Classification.csv (external dataset3)

#################################################################
1. Parsing
 - Use command line
 - Parse fradulent_emails.txt file with Tika
 - Download and set up python tika based on github.
 - Run the following command under the file that stores tika.py: 
   python tika.py parse all fradulent_emails.txt
 - Output is the result file named ‘fradulent_emails.txt_meta.json’

#################################################################
2. Pre-processing
 - Use step1_pre-processing.ipynb file.
   * Open fradulent_emails.txt_meta.txt file and delete non-email values.
 - Output is the ‘fradulent_emails_preprocessing.txt_meta.json’, which will be used in all the following steps

#################################################################
3. Separation
 - Use step2_separation.ipynb file
 - Read the 'fradulent_emails_preprocessing.txt_meta.json'
 - Parser the file and parser the ‘X-TIKA:content' part from each email
 - Output is the separate email json files for all emails under the ‘separate file’ subfolder, but all emails failed the parser will be excluded
 - All following sections will read and write all individual email json file in the ‘separate file’ subfolder

#################################################################
4. Attack type
 - Use step5_01.AttackType.ipynb
 - Function get_reconnaissance search ‘Message:Raw-Header:Status’ value and return True or False Reconnaissance attack type result
 - Function get_socialEngineering search for some keywords in email content and return True or False Social Engineering attack type result
 - Function get_Malware use spacy package to detect any phrase looks like an URL and return True or False Malware attack type result
 - Function get_Credential_Phishing use spacy package to match certain patterns in email content and return True or False Credential Phishing attack type result
 - Output is all four attack types in True or False format in the metadata of the individual email json file


#################################################################
#################################################################
5. Attack Stylometrics

#################################################################
5.1 Attacker title
 - Use step5_02.AttackerTitle.ipynb
 - Function get_title use phrase matcher in spacy package to screen sentences contain “i am”, “we are” and “my name is”, then use token pattern matcher and UPOS part-of-speech tag to filter out the potential title phrase
 - Function validate_title is used to validate the output of the title and make sure they are one of the standard title in our pre-given list
 - Output is one of the standard titles. If no match is found in either function, the output is “no title exist”

#################################################################
5.2 Urgency level 
 - Use step5_03.UrgencyOfTheAttackEmail.ipynb
 - Function urgency_level generates a text’s level 1 score and level 2 score and compares them to do classification
 - Output is 0, 1, or 2. The higher score means higher urgency

#################################################################
5.3 Date/ Time
 - Use step5_04.dataAndTime.ipynb
 - Function get_dateAndTime search the ‘creation-date’ in the metadata part in each email json file and return result in date, time format
 - Output is date and time as two separated data value

#################################################################
5.4 Attacker Offering
 - Use step5_05.AttackerOffering.ipynb
 - Function get_attackerOffering screen two different sets of keywords to identify the offer from attacker in email content
 - If both money and service keywords find a match in one email, the return will be money
 - Output is Money, Service or Unknown

#################################################################
5.5 Attacker Location
 - Use step5_06.AttackerLocation.ipynb
 - Function get_AttackerLocation breaks down location identification in two different approaches
   * If an IP address is found in the metadata, use API from ipstack.com to locate the country, region and city name
   * If an IP address is not available in the metadata, use spacy to identify the country from email content. Select only the highest frequency country and leave region and city as unknown
 - Output is AttackerLocation:Country, AttackerLocation:Region, AttackerLocation:City and AttackerLocation:Country From Content

#################################################################
5.6 Attacker Relationship
 - Use step5_07.AttackerRelationship.ipynb
 - Function Check_phrase checks for phrases like “I got your contact from”,”I got your contact through” and if found it calls function Check_friend_stranger to evaluate if the next set of words in the sentence resembles a “friend of friend” or “stranger” classification.
 - Output is “friend of friend” or “stranger” and if no classification is found we classify it as “unknown”.

#################################################################
5.7 Attacker Email Sentiment
 - Use step5_08.AttackerEmailSentiment.ipynb
 - Function get_AttackerSentiment uses TextBlob package for processing textual data and calculate sentimentPolarity and sentimentSubjectivity
 - Output are AttackerSentiment:polarity and AttackerSentiment:subjectivity scores

#################################################################
5.8 Attacker Language Style
 - Use step5_09.AttackerLanguageStyle.ipynb
 - Function get_AttackerLanguageStyle uses the SpellChecker library to find the Misspelling and string functions to check for random capitalizations.
 - To evaluate the Misspelling, the values are labelled as “Level 1” if it is less than or equal to “0.05”. “Level 2” is assigned if the value is less than or equal to “0.1” and anything above this is labelled as “Level 3”
 - To evaluate random capitalizations, the values are labelled as “Level 1” if it is less than or equal to “0.1”. “Level 2” is assigned if the value is less than or equal to “0.5” and anything above this is labelled as “Level 3”
 - Output is “Level 1”, “Level 2” or “Level 3” for both AttackerLanguageStyle:misspelling and AttackerLanguageStyle:randomCapital

#################################################################
5.9 Age Predictor
 - Use Age_predictor_files_reformatting.ipynb
   * Process raw data named blogtext.csv and fradulent_emails_preprocessing.txt_meta_v2.json (same file as fradulent_emails_preprocessing.txt_meta.json)
   * From blogtext.csv, output the formal training set: age_text_train.txt; from fradulent_emails_preprocessing.txt_meta_v2.json, output contents_for_age_predictor.txt
   * In the Mac terminal, (age_train_in_terminal.txt records detailed code) firstly train a model called en-ageClassify.bin in Age_predictor’s model folder, and do the final age prediction with the model and the testing file, contents_for_age_predictor.txt. 
   * Output from terminal codes is stored as ‘age_predictor.txt’. 
 - Use step5_10.AttackerEstimatedAge.ipynb
   * Read the predicted age value from the file ‘age_predictor.txt’ and write the data into individual email json file
   * Output is the integer value of Estimated age
  
#################################################################
5.10 Attacker IP Known as Phisher
 - Use step3_PhisherIPtable.ipynb to create a table with IP and its risk level.
   * Function get_IPList uses fradulent_emails_preprocessing.txt_meta.json extract all the IP addresses in our email dataset
   * Function get_IPList_removeDup uses IP address list to remove duplicate IP address
   * Function create_IPtable uses IP address list to search risk level of the IP address from our external phisher IP database API (https://api11.scamalytics.com/)
   * Output is 'PhisherIPTable.csv' file 
 - Use step5_11.AttackerIPKnownAsPhisher.ipynb to get ‘Attacker IP Known As Phisher’ feature
   * Function get_AttackerIPfromCSV uses 'PhisherIPTable.csv' file to get a risk level of IP if the IP address exists in our metadata of the email.
   * Output is Attacker IP Known As Phisher with value as low, medium, high, very high or unknown


#################################################################
#################################################################
6. Extra datasets

#################################################################
6.1 Disaster Dataset
 - Use step4_01.DisasterTable.ipynb to unify tables from external tsv files of Earthquake and Tsunami.
   * Function get_TsunamiTable (or get_EarthquakeTable) is used to read external tsv file
   * Function get_TStable_key (or get_EQtable_key) is used to choose specific columns to use
   * Create a pivot table based on each year. Get sum of death and injuries in a given year, sum of damages in a given year, and number of disasters in a given year.
   * Output files: disasterTable.csv, disasterTable_pivot_Year.csv
 - Use step6_01.DisasterDataset.ipynb to get three features.
   * Function get_Disaster uses an email json file to get the number of deaths and injuries, damages, and the number of disasters in the year email sent to a victim.

#################################################################
6.2 WorldBank Dataset
 - Use step4_02.WorkdBankTable.ipynb to unify tables from external xml files of GDP per capita, individual using the internet, and the age dependency ratio.
   * This file parses the xml files and saves them as a csv file. 
   * Output files: WorldBankTable.csv
 - Use step6_02.WorldBankDataset.ipynb to get three features.
   * Function get_WorldBank uses an email json file to get the GDP per capita, individual using the internet, and the age dependency in the year and the country of the email sent from the attacker.

#################################################################
6.3 MaliciousURLDataset
 - Use step4_03.MaliciousURLtable.ipynb to modify the tables from external csv files of Malicious URL. One external csv file contains IP with its level (good/bad) and the other contains classification (adult, sports, etc)
   * Get the Host URL from each table and get an average length of the URL based on the label.
   * Output file: maliciousURL.csv, URLHostClassification.csv
 - Use step6_03.MaliciousURLDataset.ipynb to get three features.
   * Function get_URLInfo uses an email json file and two external tables modified previously to get label, URL length compared to average bad URL length, and the classification if the email contains URL.

#################################################################
7. TSV file formation
 - Use step7.extractAllFeatures.ipynb
 - Read all 29 features from individual email json file
 - Write features into a tsv file, in which rows indicate email objects and all features are added in as columns
 - Output is ‘featureTable.csv’, ‘featureTable.tsv’ and ‘‘BigJsonFile.json’ file
 - Create ‘featureJson’ subfolder and put separate json files containing features of the individual emails

#################################################################    
8. For Tika Similarity
 - About preparing separate json files containing each email’s features, the second half of step7.extractAllFeatures.ipynb has done this work. 
 - jaccard.py, edit.py and cosine.py, shared by Jiarui Ou, are used to calculate similarity scores via three methods and export csv files for them.  (Codes in terminal seen tika_similarity_in_terminal.txt)
 - circle-packing-for-all.py, shared by Jiarui Ou, is used to generate a json file as the clustering result.   
 - Visualization is done by opening a (current directory’s) localhost with simplehttpserver, visiting http://localhost:8000 and accessing into circlepacking.html. 

#################################################################
9. Get similarity cluster results appended into TSV file 
 - Once we get the result file from the previous step (circle-packing-for-all.py), we save the output file and name it as “similarity method_circle.json”. (Jaccard_circle.json, edit_circle.json and cosine_circle.json)
 - Run step8.similarityToTable.ipynb with input files, Jaccard_circle.json, edit_circle.json and cosine_circle.json to add new columns, indicating the cluster which each email belongs to under specific method. 
 - Output is a csv file with all features and the clustering results, named “featureTable_withCluster.csv”.   
