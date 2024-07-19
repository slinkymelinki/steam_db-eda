# steam_db-eda
Current ongoing project using Nik Davis Steam Store Raw Data (uncleaned) dataset to tackle interesting data cleaning, normalising, and overall approach to data processing questions.   The data was acquired from polling Steam and SteamSpy APIs.

Workflow Analysis and Data Exploration of Steam Game Data: Data Cleaning, Normalization, and Visualization
(IN-PROGRESS)
By Lewis Freeston


 


Overview
This report outlines the comprehensive process undertaken to clean, normalize, and prepare a Steam game database for analysis and eventual visualisation. The process was carried out in several steps: initial cleaning, normalization, and transformation of the data, with a focus on ensuring data integrity and usability for future analysis in MySQL, PowerBI.
Programs
•	Excel, VBA Editor, Power Query, MySQL, PowerBI.

1	Initial Data Cleaning
Upon initial inspection of the downloaded archive of Steam DB information there are three different files.
•	app_list
o	Table identifying games and their ID. 
•	steam_app_data
o	Table identifying specific metrics of each game, including ID, price, controller support, developer metrics, price information and release date information.
•	steamspy_data
o	Table identifying specific player centric metrics of steam games including userscore, reviews both positive and negative, concurrent players etc.
As an initial focus, steam_app_data was to contain the most relevant information for a ‘master’ file to begin working off. The downside to choosing steam_app_data is that I imagine due to the method of data extraction, the special characters and formatting of text-based information is incredibly complex and will provide an excellent overview of my current skills, and ability to adapt to complex data transformation.
Below is a workflow ‘diary’ of sorts on how I approached the file, with examples of functions and ideas of the eventual database schema, with considerations made for further analysis using MySQL specifically, with a focus on PowerBI visualisations on the back end of the project.
1.1	Name Cleaning
•	Created a new column name_clean and used the formula =TRIM(UPPER(B2)) to format game titles properly.
 
•	Filtered out and deleted blank records.
•	Identified and removed records with corrupted text, particularly those with Chinese and Russian characters which converted incorrectly. Those records with primarily null values were removed as they were not statistically relevant for my questions.
1.2 Duplicates Removal
•	Removed duplicates across the dataset using Ctrl+A twice to select all available ranges.
•	Investigated NULL values and filtered out irrelevant data types, such as regulatory literature entries which seem to be an export error from the original data scraping.
2. Data Transformation and Normalization
The current headers were copied and =TRANSPOSE(array) to another sheet to annotate each column and provide a description, and justification for keeping or removing the column.
2.1 Column Review and Decisions
Column Name	Action Taken	Reasoning
type	Maybe	Repetitive data with no other types detected. If other fields were present, the data could be normalised in another table.
name	Kept	Essential for identifying games.
steam_appid	Kept	Key identifier for games.
required_age	Kept	To analyze age restrictions.
is_free	Kept	To analyze free vs. paid games.
controller_support	Converted to Boolean	Simplified to indicate presence or absence of controller support.
dlc	Removed	Not useful without corresponding data.
detailed_description	Removed	Too verbose.
about_the_game	Removed	Duplicate of detailed_description.
short_description	Removed	Duplicate of detailed_description.
fullgame	Removed	All null values.
supported_languages	Maybe	Further consideration needed.
header_image	Removed	URL not needed for analysis.
website	Removed	URL not needed for analysis.
pc_requirements	Removed	Too verbose.
mac_requirements	Removed	Too verbose.
linux_requirements	Removed	Too verbose.
legal_notice	Removed	Too verbose.
drm_notice	Kept	To analyze the impact of DRM on game performance/price.
developers	Kept	For aggregation and analysis.
publishers	Kept	For aggregation and analysis.
demos	Removed	Not useful without corresponding data.
price_overview	Kept	Essential for price analysis.
packages	Removed	Not useful without corresponding data.
package_groups	Removed	Not useful without corresponding data.
platforms	Kept	Analyzed and normalized.
metacritic	Kept	Cleaned to retrieve score out of 100.
reviews	Removed	Too verbose.
categories	Kept	Separated into single-player and multiplayer games.
genres	Kept	Normalized and assigned genres.
screenshots	Removed	URL not needed for analysis.
movies	Removed	URL not needed for analysis.
recommendations	Kept	For ranking and interaction analysis.
achievements	Kept	Numbers for analysis.
release_date	Kept	To track release dates.
support_info	Removed	URL not needed for analysis.
background	Removed	URL not needed for analysis.
content_descriptors	Removed	Too granular for current scope.
2.2 Standardization and Cleaning
2.2a Name
•	Initially converted game titles using =TRIM(PROPER(A2)) however this led to conversion artefacts including decapitalised roman numerals, and strange formatting for games with non-standard names. Therefore, the decision was made to convert using =UPPER to consolidate the format and readability of the names.
o	Converted game titles to uppercase using UPPER to handle anomalies like Roman numerals.
o	In a few cases CLEAN was used to remove non-printable characters by integrating with UPPER. =CLEAN(UPPER(A2))
 
•	Created another column to further screen game titles for additional characters which appear to be characters and symbols in languages such as Chinese, Russian and Thai
•	Created a VBA function to remove special characters and clean corrupted text in various columns.
o	This function below can be amended by adding or removing characters to the allowedChars class. This will exclude all characters outside of normal special characters and alphanumeric values. This is called in the cell using =RemoveSpecialChars(A2). 
	Function RemoveSpecialChars(text As String) As String Dim i As Integer Dim char As String Dim result As String Dim allowedChars As String ' Define allowed characters (alphanumeric and specific special characters) allowedChars = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789!- .,/()[]*#"
' Loop through each character in the input text For i = 1 To Len(text) char = Mid(text, i, 1) ' Check if the character is in the list of allowed characters If InStr(allowedChars, char) > 0 Then result = result & char End If Next i
RemoveSpecialChars = result End Function
•	Filtered and manually checked records with Chinese or Russian characters for integrity. Those with either just special characters as titles were removed via the function and were deleted from the dataset.
o	Additionally, records with just parentheses and square brackets remaining i.e. ‘()’ and ‘[]’ were manually screened using filters due to a low volume of entries.
2.2b Steam ID
Steam ID will be retained as in another table, these are keyed against the game title which will improve performance and allow demonstrations with INNER and OUTER joins in MySQL.
2.2c Required Age
Required Age will be retained as it already in a integer format. Those will NULL values were filled using Ctrl+Enter with ‘0’ in the field.
2.2d Is Free
Is Free will be retained as it will provide a boolean True or False as to whether the game is free or has an associated price.
2.2e Control Support
Control Support will be retained as it will provide a True where applicable, the the remainder will be NULL values allowed in SQL when importing.
2.2 Supported Languages
•	Due to formatting of cell, all strings contained within <> can be removed using * as a wildcard (<>). When efforts were made to remove the * from cells, I had to use a string literal in the form of ¬ to make the character to the right of the ¬ literal.
o	All values are now in CSV and can be broken out using text to columns.
•	Counted the number of supported languages and considered creating a separate table for unique language combinations to reduce redundancy.
o	=LEN(I2)-LEN(SUBSTITUTE(I2,",",""))+1 
	This formula counts the fields separated by commas, allowing for a total number of supported languages to be counted and ranked. 29 appears to be the maximum for supported languages.
•	I can foresee issues with querying with 29 additional columns which for the most part will be empty, as only those games with significant support will be utilising the full complement of cells 1 through to 29.
o	Therefore, I believe the data should be broken out and normalised into a series of tables which are interrelated.
Languages Table
•	LanguageID 
o	(Primary Key)
•	LanguageName
LanguageID	LanguageName
1	English
2	German
3	Traditional Chinese
4	Portuguese
5	Russian
6	French
7	Spanish
8	Italian
•	LanguageCombinations Table:
	CombinationID 
	(Primary Key)
	CombinationName 
	(Readable name or description of the combination)
CombinationID	CombinationName
1	English, German, Traditional Chinese, Portuguese, Russian
2	French, Spanish, Italian
CombinationLanguages Table:
CombinationID	LanguageID
1	1
1	2
1	3
1	4
1	5
2	6
2	7
2	8
steam_app_data example
name	example_details	CombinationID
1	Details for Entry 1	1
2	Details for Entry 2	2
3	Details for Entry 3	1
Only CombinationID would be stored in the original steam file and would be far more condensed than 29 additional columns bloating the file. This approach would:
•	Efficiency: Reduces redundancy by storing unique language combinations once.
•	Data Integrity: Ensures that changes in language combinations are easily manageable.
•	Scalability: Simplifies the process of adding or modifying entries and their associated languages.
2.4 DRM
•	Consolidated similar DRM names and created a clean list for analysis. 
o	DRM Free! converted into NULL.
•	Before
 
•	After
 
2.5 Developers and Publishers
•	Cleaned using VBA RemoveSpecialChars function and formulas to ensure proper formatting and removed special characters. 
o	Amended function to include [] to remove these from the formatting of the developer and publisher names.
o	Identified a large proportion of games without an assigned developer, but a publisher so decided to leave these as null as they did contain price information which was of higher importance. 
	Publisher was entirely resolved using =TRIM(PROPER(RemoveSpecChars(B2)))
•	This will ideally be normalised and broken out into its own tables with a unique key identifying each publisher, and developer.
2.6 Price Overview
•	Extracted relevant price information and normalized currency values. 99% of records are in GBP format.
o	{'currency': 'GBP', 'initial': 5999, 'final': 5999, 'discount_percent': 0, 'initial_formatted': '', 'final_formatted': '$59.99'}
o	Cleaned using a series of find and replaces until data remained in CSV format.
 
•	Identified that only those with discount greater than 0% had a value for the initial_price_formatting field. As no information was included as to when specifically, this information was gathered, I will be removing the discount and formatting columns as they are redundant.
•	Ensured correct formatting and handling of special characters related to currency.
•	Currency will be broken out into its own table to prevent data redundancy.
2.7 Platforms
•	Normalized platform combinations and proposed a separate table for OS combinations.
 
•	OS table
o	os_id
o	os_name
•	OS Combinations table
o	os_combination_id
o	combination_name
•	CombinationsOS Table
o	os_combination_id
o	os_id
Only include CombinationsOS in main file and other tables will supplement the combination of OS.
2.8 Metacritic Scores
•	Cleaned to retrieve only the score out of 100. 
o	As score is out of 100. Only the first 13 characters of the string were required to perform the cleaning process. This was helpful as all other data in the cell was a large, unique hyperlink which would have proved to be difficult to remove.
o	Once =LEFT(AD24,13) was used. {'score': was replaced with nothing and filters were checked to verify that only integer values remained.
2.8 Categories and Genre
•	Similar to Languages, this contained a long-complicated string which was systematically cleaned using find and replace to convert:
[{'id': 2, 'description': 'Single-player'}, {'id': 22, 'description': 'Steam Achievements'}, {'id': 18, 'description': 'Partial Controller Support'}, {'id': 23, 'description': 'Steam Cloud'}]
•	Into a CSV format which could be broken out into another sheet and recombined to create a catergory_key table and genre_key table
2.9 Recommendations and Achievements
•	Simplified by extracting relevant numerical data. 
o	Recommendations were cleaned by simply substituting {'total': and } from the column to leave just integers.
o	Achievements utilised the =LEFT formula for 14 characters as the maximum achievement limit for steam is 1000’s. Cleaned using find and replace.
2.10 Release Date
•	Standardized dates, adding long and short month columns for flexibility. 
o	Added a day, long month, short month, year column.
o	text to columns was used to separate out values in another sheet and minor adjustments were made using filters to identify mismatched entries and were manually processed and moved to the relevant columns.

