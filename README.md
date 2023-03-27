# Overview

In order to come up with recommendations for Microsoft in connection with its interest in entering the film market and producing original content, I made use of five data source to recommend that a) Microsoft should spend money, but with caution, in order to achieve higher profits, b) Microsoft should target the documentary, comedy and drama genres and the ratings categories of PG-13, PG and R, and c) Microsoft should focus on the movies and actors highlighted in the presentation who did well in terms of box office sales and fan ratings.

# Business Problem
My task was to come up with 3 recommendations for Microsoft in connection with its interest in entering the film market and producing original content.

# Data Understanding
we were provided with data from several different sources: Box Office Mojo, Rotten Tomatoes, the IMDB, TMDB and TN.  The data sets were of different sorts and contained different information.  IMDB was the only SQL/relational database; all the others were either comma or tab separated formats.  The concentration of the IMDB database was on the people associated with the movie.  The concentration of the Box office Mojo and TN data were on the financial aspects of the movies.  The concentration of the Rotten Tomatoes and TMDB data were on the fan ratings and other specifics of the movies.



# See the distribution of films by studio
df2studios= df2['studio'].value_counts().head(20)
​
df2studios
df2studios
IFC       166
Uni.      147
WB        140
Fox       136
Magn.     136
SPC       123
Sony      110
BV        106
LGF       103
Par.      101
Eros       89
Wein.      77
CL         74
Strand     68
FoxS       67
RAtt.      66
KL         62
Focus      60
WGUSA      58
CJ         56

sum(df2['domestic_gross'])



# Find the most popular genres.
q = pd.read_sql("""
SELECT *
  FROM movie_basics;
""", conn)
q2 = q.groupby('genres').count()
q2.sort_values('primary_title', ascending = False).head(25)

q2_1 = q2.sort_values('movie_id', ascending = False).head(7)
q2_1

q2_2 = q2_1.loc[:, 'movie_id']

q2_2.plot(kind = 'bar')

# Find out what genres get the highest ratings.
q3 = """SELECT mb.genres, mr.averagerating, mr.numvotes
        FROM movie_basics as mb
        JOIN movie_ratings as mr
            ON mb.movie_id = mr.movie_id
        GROUP BY mb.genres
        ORDER BY mr.averagerating DESC
        LIMIT 30"""
pd.read_sql(q3,conn)

genres	averagerating	numvotes
0	Documentary,Family,Romance	9.7	25
1	Comedy,Documentary,Sport	9.7	22
2	Comedy,Documentary,Fantasy	9.4	5
3	Documentary,Family,Musical	9.3	19
4	History,Sport	9.2	5


movie_id	primary_title	original_title	start_year	runtime_minutes
genres					
Documentary	32185	32185	32185	32185	24672
Drama	21486	21486	21486	21486	15725
Comedy	9177	9177	9177	9177	6413
Horror	4372	4372	4372	4372	2975
Comedy,Drama	3519	3519	3519	3519	3163
Thriller	3046	3046	3046	3046	1924
Action	2219	2219	2219	2219	1153


df7 = pd.read_csv("C:\\Users\\jmark\\Documents\\Flatiron\\phase_1_project\\rt.movie_info.tsv", '\t')

df7['rating'].unique()

df7['rating'].value_counts()

R        521
NR       503
PG       240
PG-13    235
G         57
NC17       1
Name: rating, dtype: int64

df3



# There is some correlation between production budget and sales, especially internationally, but many lower budget movies have higher sales as well.
df3.plot('domestic_gross', 'worldwide_gross', kind='scatter')

df9 =df8.groupby('rating').mean('box_office')
df9

# Below, it's shown the top four most popular ratings, in order, are PG-13, PG, R and G.  PG-13 has almost 3x as much sales as R.

rating		
G	1368.400000	7.402788e+06
NC17	1567.000000	1.260219e+06
NR	1160.222222	6.376923e+05
PG	1071.657895	5.289280e+07
PG-13	1053.506494	6.872359e+07
R	921.019048	2.394827e+07

df10.plot(kind = 'bar')


# Highest rated movies.
q7 = """SELECT mb.primary_title, mr.averagerating, mr.numvotes
        FROM movie_basics as mb
        JOIN movie_ratings as mr
            ON mb.movie_id = mr.movie_id
        ORDER BY mr.averagerating DESC
        LIMIT 30"""
pd.read_sql(q7,conn)

primary_title	averagerating	numvotes
0	Exteriores: Mulheres Brasileiras na Diplomacia	10.0	5
1	The Dark Knight: The Ballad of the N Word	10.0	5
2	Freeing Bernie Baran	10.0	5
3	Hercule contre Hermès	10.0	5
4	I Was Born Yesterday!	10.0	6

# Find out the professionals/actors who have been voted most on by fans.
q8 = """SELECT p.primary_profession, p.primary_name, mr.averagerating, mr.numvotes
        FROM movie_ratings as mr
        JOIN known_for as kf
            ON mr.movie_id = kf.movie_id
        JOIN persons as p
            ON kf.person_id = p.person_id
        WHERE (primary_profession = 'actress' OR primary_profession = 'actor')
        ORDER BY mr.numvotes DESC
        LIMIT 30"""
pd.read_sql(q8, conn)


primary_profession	primary_name	averagerating	numvotes
0	actor	Mark Fleischmann	8.8	1841066
1	actress	Claire Geare	8.8	1841066
2	actor	Wade Williams	8.4	1387769
3	actor	Oliver Cotton	8.4	1387769
4	actor	Cameron Jack	8.4	1387769

# Find out the actors who have been voted highest on by fans.
q8 = """SELECT p.primary_profession, p.primary_name, mr.averagerating, mr.numvotes
        FROM movie_ratings as mr
        JOIN known_for as kf
            ON mr.movie_id = kf.movie_id
        JOIN persons as p
            ON kf.person_id = p.person_id
        WHERE (mr.numvotes > 50) AND (primary_profession = 'actress' OR primary_profession = 'actor')
        ORDER BY mr. averagerating DESC, mr.numvotes DESC
        LIMIT 30"""
pd.read_sql(q8, conn)

	primary_profession	primary_name	averagerating	numvotes
0	actor	Loki	9.9	417
1	actor	Chris Scagos	9.7	5600
2	actor	John Luder	9.7	5600

3	actor	Rafal Zawierucha	9.7	5600
4	actress	Jennifer Churchich	9.7	5600

# First step to show actors/actresses in highest grossing movies: Join SQL IMDB tables in order to have actresses and movie in same table.
q10 = """SELECT mb.primary_title, p.primary_profession, p.primary_name, mr.averagerating, mr.numvotes
        FROM movie_ratings as mr
        JOIN movie_basics as mb
            ON mr.movie_id = mb.movie_id
        JOIN known_for as kf
            ON mr.movie_id = kf.movie_id
        JOIN persons as p
            ON kf.person_id = p.person_id
        WHERE (mr.numvotes > 50) AND (primary_profession = 'actress' OR primary_profession = 'actor')
        ORDER BY mr. averagerating DESC, mr.numvotes DESC"""
df12 = pd.read_sql(q10, conn)

df12.set_index('primary_title', inplace=True)

df3.set_index('movie', inplace=True)

# Last step to show actors/actresses in highest grossing movies: Join two data frames to add sales figures to actor/movie dataframe.
df_actorsSales = df3.join(df12, how = "inner")



actorsSalesSorted = df_actorsSales.sort_values('worldwide_gross', ascending = False).head(30)
df_actorsSalesSorted 

id	release_date	production_budget	domestic_gross	worldwide_gross	profits	primary_profession	primary_name	averagerating	numvotes
Avengers: Infinity War	7	Apr 27, 2018	300000000	678815482	2.048134e+09	1.748134e+09	actor	Winston Duke	8.5	670926
Avengers: Infinity War	7	Apr 27, 2018	300000000	678815482	2.048134e+09	1.748134e+09	actor	Ethan Dizon	8.5	67092

df_actorsSalesSorted.iloc[:,4:]

	worldwide_gross	profits	primary_profession	primary_name	averagerating	numvotes
Avengers: Infinity War	2.048134e+09	1.748134e+09	actor	Winston Duke	8.5	670926
Avengers: Infinity War	2.048134e+09	1.748134e+09	actor	Ethan Dizon	8.5	670926

# Check for seasonality in vote_average
months = []
for i in df3['release_date']:
    newmonth = i[0:3]
    months.append(newmonth)   
months

df3['releaseMonth']=months
df3_month = df3.loc[:, ("worldwide_gross",'releaseMonth')]

df3_month.groupby('releaseMonth').mean().plot(kind='bar')

lot:xlabel='releaseMonth'> 
No seasonality given low grossing averages in Aug, Jan, high in November, May.

# estimate size of market by year, at least tens of billions.


# Conclusion¶
Microsoft should follow these 3 recommedations: a) Microsoft should spend money, but with caution, in order to achieve higher profits, b) Microsoft should target the documentary, comedy and drama genres and the ratings categories of PG-13, PG and R, and c) Microsoft should focus on the movies and actors highlighted in the presentation who did well in terms of box office sales and fan ratings
